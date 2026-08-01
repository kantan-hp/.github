# Security Policy

Thanks for helping keep Kantan and the people who rely on it safe. This is the
default policy for every kantan-hp repository; a repo may publish its own
`SECURITY.md` that takes precedence.

## Reporting a vulnerability

**Please don't open a public issue, pull request, or discussion for a security problem** —
that discloses it to everyone before there's a fix.

Report it privately, either way:

- **GitHub** — use **Report a vulnerability** on the affected repo's **Security ▸ Advisories**
  tab (where private reporting is enabled). This keeps the report and the fix in one place.
- **Email** — **security@kantan-hp.app**.

Helpful to include:

- What the issue is and what an attacker could do with it.
- Steps to reproduce, or a proof of concept.
- The affected repo, app version / commit, and platform (e.g. iOS version, device).

## What to expect

We're a small team, so this is best-effort rather than an SLA: we aim to **acknowledge a
report within a few business days**, keep you updated on remediation, and credit you if
you'd like once a fix ships. Please give us a reasonable window to fix an issue before any
public disclosure — we'll work with you on timing (coordinated disclosure).

## Scope

Kantan is a free blog that users run and edit themselves. Anything that threatens that —
unexpected data collection or exfiltration, account or OAuth takeover, secret exposure,
XSS or injection that could run on an editor's or visitor's device, or a way to take over
a site's Cloudflare Pages project — is squarely in scope alongside the usual classes
(auth, injection, secret exposure, RCE).

The public web starter lives in
[`kantan-hp`](https://github.com/kantan-hp/kantan-hp); the control panel and the mobile
apps (iOS / Android) live in adjacent repositories. Report issues in any of them to the
address above — you don't need to know which repo is at fault.
