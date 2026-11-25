---
title: "The Work Behind This Blog: Self-Employed at techz.at"
date: 2026-11-25
tags: ["techz", "self-employment", "linux", "automation", "infrastructure"]
description: "I'm self-employed at techz.at, helping businesses with Linux infrastructure, automation, and planned maintenance. Here is how that work connects to this blog."
draft: false
---

Most articles here start with an operational question: can a backup actually be restored, how should a configuration change reach production, or what happens when a service keeps restarting?

There is a business side to those questions too. I am self-employed at **[techz.at](https://techz.at/)**, where I help businesses automate their infrastructure and recurring processes. This post connects that work with the technical subjects I write about on emr3.me.

{{< figure src="featured.svg" alt="An engineering workbench connects a Linux terminal to an automated workflow, beside the words The work behind the blog and techz.at." >}}

## From Linux engineering to business operations

I am Emre Hayta, a Linux and cloud engineer based in Upper Austria. I have worked professionally in this field since 2014. Through techz.at, I bring that background to small and medium-sized businesses that need their systems to be understandable, maintainable, and less dependent on manual work.

That can mean taking over an existing Linux environment, turning server configuration into code, or connecting tools that currently depend on someone copying information between them.

The starting point is understanding the work: what runs today, who relies on it, and which problems are worth addressing first. A useful project leaves the business with something its team or a future service provider can operate.

## Infrastructure that can be rebuilt

The [infrastructure automation work at techz.at](https://techz.at/ansible-automation.html) covers building or taking over server environments and describing their configuration in code. Linux, Ansible, and Docker are central tools in that work.

The deliverable extends beyond a running server. Configuration, access, dependencies, backup procedures, and the handover all matter. A system is easier to maintain when the next engineer can inspect how it was built and understand how to change it.

That is also why this blog covers [controlled Ansible rollouts]({{< ref "ansible-controlled-rollouts" >}}) and [server handovers]({{< ref "linux-server-takeover" >}}). Those articles explore the engineering decisions behind operating an environment responsibly.

## Automation beyond the server

Some of the most repetitive work happens between applications: forwarding messages, saving attachments, moving information into another system, or preparing recurring reports.

For [process automation at techz.at](https://techz.at/n8n-automation.html), I work with self-hosted n8n, scripts, and APIs according to what the workflow needs. The platform underneath also needs attention: deployment, TLS, backups, monitoring, and documentation are part of keeping an automation useful.

Consider a simple illustrative workflow: an incoming document needs to be identified, saved consistently, and passed to the right person or system. Before automating it, we need to define what happens when the document is incomplete or does not fit the expected format. Exceptions need a visible route to a person.

A self-hosted workflow can still send information to external services through its integrations. Understanding that data flow belongs in the design and handover.

## Planned maintenance after the project

Systems continue to change after setup. Updates arrive, credentials rotate, certificates renew, and data grows.

The [ongoing maintenance offered through techz.at](https://techz.at/it-betreuung-kmu/) gives businesses an option for planned upkeep after the environment has been understood and taken over. Scope and responsibilities need to be clear, including what is checked and how follow-up work is handled.

This connects directly to the [restore-testing article]({{< ref "backup-restore-testing" >}}): a successful backup notification answers a narrower question than a demonstrated recovery. Maintenance needs useful evidence of the system's condition.

## Where the blog fits

emr3.me is where I share practical engineering notes and explanations. techz.at is where businesses can find my services and discuss their own environment.

The connection is straightforward: the blog lets you see how I approach technical problems before we talk about a project. You can use the articles independently, or bring a specific infrastructure or workflow problem to a conversation.

If your team is dealing with undocumented Linux servers, repeated manual configuration, or work that keeps getting copied between systems, visit **[techz.at](https://techz.at/)** to explore the services and get in touch with me directly.
