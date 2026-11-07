---
title: "The Certificate Renewed. Why Is the Website Still Serving the Old One?"
date: 2026-10-02
tags: ["tls", "linux", "nginx", "monitoring", "operations"]
description: "Follow TLS renewal from ACME validation to the certificate users receive: test Certbot, reload the right service, and monitor the public endpoint."
draft: false
---

The certificate on disk has a new expiry date. The browser still reports the old one.

That can happen because certificate renewal and certificate deployment are separate operations. The ACME client obtains new files. A web server, load balancer, ingress, or CDN must then present the intended certificate to users.

Troubleshooting becomes easier when you follow that entire path.

{{< figure src="featured.svg" alt="Certificate files on a server pass through a reload step to the public TLS endpoint, where a browser checks the certificate actually served." >}}

## Locate the TLS termination point

First identify which component handles the public connection. A site's Nginx backend may never present a certificate to the browser if a CDN terminates TLS in front of it.

Draw the actual path, for example:

```text
Browser -> CDN -> load balancer -> application host
```

There may be separate certificates for the browser-to-CDN connection and the CDN-to-origin connection. Renewing one does not update the other.

Record the public hostname, relevant IPv4 and IPv6 destinations, and each termination point. During a migration, different addresses can accidentally reach different servers.

## Inspect what the endpoint serves

On a machine with OpenSSL, inspect the certificate returned for a hostname:

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null \
  | openssl x509 -noout -subject -issuer -dates -fingerprint -sha256
```

Replace `example.com` with the real hostname. The `-servername` option sends SNI so a shared listener can select the intended certificate.

This pipeline is an inspection aid. It prints certificate details and does not by itself establish trust-chain validity or hostname validity. Its final exit status may also hide an earlier command's failure.

For a deliberate verification check, with a correctly configured local trust store, use:

```bash
openssl s_client -connect example.com:443 -servername example.com \
  -verify_hostname example.com -verify_return_error </dev/null
```

Review the verification result and connection errors. To inspect a specific origin address, use that address in `-connect` while retaining the public hostname in `-servername` and `-verify_hostname`. That only tests the endpoint you selected, not every node in a load-balanced pool.

The OpenSSL [s_client reference](https://docs.openssl.org/3.5/man1/openssl-s_client/) describes these options.

## Compare the configured file

For a Certbot-managed certificate, list the local certificate records:

```bash
sudo certbot certificates
```

Use the reported certificate path to inspect the on-disk leaf certificate:

```bash
sudo openssl x509 -in /etc/letsencrypt/live/example.com/fullchain.pem \
  -noout -subject -issuer -dates -fingerprint -sha256
```

The actual certificate directory may have a different name. Use the reported path rather than assuming it matches the hostname exactly.

A different fingerprint at the public endpoint is a clue, not automatic proof of failure: CDNs and other managed terminators may intentionally use their own certificates. Compare the files and endpoint that are supposed to match.

For a host-managed server, check which certificate paths its effective configuration references. An old copied file or an outdated container mount can explain why a successful renewal never reaches the listener.

## Test the renewal mechanism

For an existing Certbot installation:

```bash
sudo certbot renew --dry-run
```

This exercises renewal against a staging service without saving a production replacement certificate. Depending on the authenticator, it can still interact with web server configuration or hooks, so run it in an appropriate window and understand the installed setup.

Check the challenge method. HTTP validation needs a working inbound path for the challenge. DNS validation needs credentials and the ability to update the correct DNS zone. A DNS change, proxy rule, or credential rotation can break a previously working process.

Do not assume a particular timer name. Inspect how this installation schedules renewals; packages, snaps, and other installation methods can differ. Avoid creating an extra cron job before checking the existing scheduler.

## Deploy only after successful renewal

For a conventional host-managed Nginx deployment, a deploy hook might validate configuration and reload the service:

```sh
#!/bin/sh
set -eu
/usr/sbin/nginx -t
/usr/bin/systemctl reload nginx
```

Confirm executable paths on the target system. Install it as a root-owned executable in Certbot's deploy-hook directory only when this matches your architecture. A containerised proxy or external load balancer needs a different deployment mechanism.

A configuration-test failure stops this hook before reload. Record and alert on that failure rather than treating the renewed file as a completed deployment.

Deploy hooks run after successful renewal. A normal `certbot renew` exit status of zero can also mean no certificate was due, so it is not evidence that a new certificate was deployed. Dry runs do not run deploy hooks by default; `--run-deploy-hooks` changes that behaviour and can affect the running service. See the [Certbot renewal guide](https://eff-certbot.readthedocs.io/en/stable/using.html#renewing-certificates) before testing hooks.

## Monitor the user-facing result

Run an independent HTTPS check that validates hostname, trust, and expiry on the public service. Choose warning windows that leave enough time for your support process to respond, and account for the certificate lifetimes actually in use.

Keep internal checks for renewal jobs and deployment hooks as well. The external check catches a stale listener; internal evidence helps locate the failed step.

For multiple termination nodes, verify that every relevant node receives updates. One successful request can miss a stale member of the pool. Include IPv6 where published, and inspect both origin and edge certificates when both matter.

The operational chain is complete when the intended endpoint serves a valid certificate to clients. Keeping that distinction in your [monitoring plan]({{< ref "monitoring-small-linux-fleet" >}}) prevents a green renewal job from concealing a broken deployment.
