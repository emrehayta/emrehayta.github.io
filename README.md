# emr3.me — Emre Hayta's Engineering Blog

Source code and English articles for [emr3.me](https://emr3.me/), my blog about Linux, cloud infrastructure, automation, and operating reliable systems.

I'm **Emre Hayta**, a Linux and cloud engineer based in Upper Austria. I am **self-employed at [techz.at](https://techz.at/)**, where I help businesses automate their infrastructure and recurring processes.

My work through [techz.at](https://techz.at/) includes Linux infrastructure, Ansible automation, Docker deployments, self-hosted workflows, and planned maintenance. The blog shares practical engineering knowledge alongside that work.

## Work with me

- [Infrastructure automation at techz.at](https://techz.at/ansible-automation.html)
- [Process automation with n8n at techz.at](https://techz.at/n8n-automation.html)
- [Ongoing IT maintenance at techz.at](https://techz.at/it-betreuung-kmu/)

Read more about the connection between my work and the blog: [The Work Behind This Blog: Self-Employed at techz.at](https://emr3.me/posts/self-employed-techz/).

## Run locally

The site uses Hugo and the Blowfish theme, with theme sources included in this repository. The current deployment uses **Hugo 0.102.3 Extended**.

Place the matching Hugo executable at `tools/hugo`, then run:

```bash
./hugo-old.sh version
./hugo-old.sh server -D
```

Open <http://localhost:1313/>. The `tools/` directory is ignored by Git; the binary is installed locally rather than committed.

## Content and publishing

Articles live in `content/posts/<slug>/index.md`. Each article's images live alongside it, usually as `featured.png` or `featured.svg`.

Production builds exclude drafts and future-dated posts. Pushing to `main` starts the GitHub Pages deployment configured in [.github/workflows/hugo.yml](.github/workflows/hugo.yml).

## Theme

Built with [Blowfish](https://blowfish.page/) by Nuno Coração. See [LICENSE](LICENSE) for the repository's existing license.
