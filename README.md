# kantan-hp/.github

Org-level defaults for Kantan (かんたん) — a free, simple blog, coming to the web, iOS,
and Android. Hosts the **reusable security workflow** that every active repo calls
through a thin caller instead of maintaining its own copy, plus the **community health
defaults** GitHub applies to every repo in the org.

## Community health defaults

Because this repo is **public**, GitHub uses the files below as the default for any
repo in the org — public or private — that doesn't ship its own copy. A repo can
override any of them by committing its own file of the same type.

| File | Applies as |
| --- | --- |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) | Contributor Covenant 2.1; conduct reports go to `security@kantan-hp.app` |
| [`SECURITY.md`](SECURITY.md) | Default security policy; private reporting via `security@kantan-hp.app` + advisories |
| [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md) | Pre-fills the PR description in every repo |
| [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE) | 🐞 Bug report + 💡 Idea / feature request issue forms, plus a chooser `config.yml` (security reports are deflected to `security@kantan-hp.app`) |

These show up in each repo's community profile as *"inherited from the kantan-hp/.github
repository."* Resolution order GitHub uses when a repo lacks its own copy: the repo's
`.github/` folder → repo root → repo `docs/`.

## Reusable security backstop

[`.github/workflows/security.yml`](.github/workflows/security.yml) is a
`workflow_call` workflow with per-scanner toggles. Each repo opts into only the
scanners it needs:

| Scanner | Input | Where it's used |
| --- | --- | --- |
| **gitleaks** (secret scan) | `gitleaks` (default `true`) | every repo |
| **Semgrep OSS** (JS/TS SAST) | `semgrep` | `kantan-hp` (Astro/TS web), the Cloudflare Worker panel, other app-logic repos |
| **mobsfscan** (mobile SAST) | `mobsfscan` | the iOS and Android apps |
| &nbsp;&nbsp;↳ SARIF → code scanning | `mobsfscan_sarif` (default `true`) | uploads on public mobile repos; auto-skips elsewhere |
| **actionlint** (workflow lint) | `actionlint` | privileged CI repos, and this repo's own `self-scan.yml` |

All scans run on GitHub-hosted `ubuntu-latest` — free for public repos, no runner fleet
to manage, and none of the scanners need repo secrets.

## How repos call it

This repo is **public**, so any repo — public or private — can call the reusable
workflow. Each consuming repo has a thin caller, e.g. `.github/workflows/security.yml`:

```yaml
# .github/workflows/security.yml in a consuming repo
name: Security
on:
  push: { branches: [main] }
  pull_request: {}
jobs:
  security:
    uses: kantan-hp/.github/.github/workflows/security.yml@main
    with:
      gitleaks: true
      semgrep: true   # only on JS/TS app repos
```

### Mobile repos: grant `security-events: write`

The `mobsfscan` job uploads its SARIF to code scanning, which needs
`security-events: write`. A reusable workflow **cannot elevate beyond the token the
caller passes**, and GitHub validates a called workflow's requested permissions at
startup — before any `if:` skips a job — so the central workflow deliberately requests
**no** elevated permission of its own (otherwise every non-mobile caller would fail to
start). Mobile callers therefore grant the scope on the calling job:

```yaml
jobs:
  security:
    permissions:
      contents: read
      security-events: write
    uses: kantan-hp/.github/.github/workflows/security.yml@main
    with:
      gitleaks: true
      mobsfscan: true
```

The upload auto-skips on private repos (code scanning needs GitHub Advanced Security)
and on fork PRs (read-only token), so the grant is a harmless no-op there.

## Notes

- **This repo scans itself** via [`self-scan.yml`](.github/workflows/self-scan.yml)
  (gitleaks + actionlint). It references `@main` rather than a local `./` path on
  purpose: a local reusable reference here produced a zero-job `startup_failure`,
  whereas `@main` runs reliably.
- **Dependency updates are not centralizable** via `workflow_call`; each repo commits
  its own `.github/dependabot.yml` (templated, not shared).
- Per-repo **build/test gates** (e.g. `npm run check`, `wrangler test`, `swift test`)
  stay in each repo — they genuinely differ.
- Follow-up hardening: SHA-pin the third-party actions/images referenced here.
