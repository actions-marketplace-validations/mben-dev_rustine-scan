# Rustine security scan

Analyses your repository on the GitHub runner: secrets written in the code or left in the
git history, risky code (SQL built by hand, HTML injected unescaped, routes without a
check…), packages with known vulnerabilities and risky configuration. The same rules as an
analysis on [rustine.io](https://rustine.io).

Your code is never sent. Only the list of packages (ecosystem, name, version) goes to
Rustine to compare it with known vulnerabilities.

```yaml
name: Security
on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  security-events: write # the results in the Security tab

jobs:
  rustine:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0 # the history, for secrets committed then removed
      - uses: mben-dev/rustine-scan@v1
        with:
          fail-on: high
```

The problems show as annotations on the pull request's lines, in the Security tab, and in
the job summary. The job fails when one is at `fail-on` or above.

## Inputs

| Input | Default | |
|---|---|---|
| `fail-on` | `high` | `critical`, `high`, `medium`, `low` or `none` (never fail) |
| `path` | `.` | the folder to analyse, for a monorepo |
| `api-key` | | a Rustine API key, from a secret: lifts the limit of 30 known-vulnerability checks an hour |
| `lang` | `en` | `en` or `fr` |
| `sarif` | `true` | upload to code scanning |
| `version` | `0` | the version of the `rustine` CLI |

Outputs: `score`, `findings`, `sarif-file`.

Code scanning is free on public repositories. On a private one it needs GitHub Advanced
Security: without it the upload step fails without failing the job, and the job summary
still holds the results.

To fix a problem, run `npx rustine scan` then `npx rustine fix <n> --copy` on your machine,
and paste the prompt into your coding agent.

---

**En français** : l'analyse de dépôt de Rustine, sur le runner GitHub. Ton code n'est jamais
envoyé, seule la liste des paquets part pour les failles connues. `lang: fr` donne le rapport
en français ; `fetch-depth: 0` permet de lire l'historique.

The action is MIT licensed. The `rustine` CLI it runs has [its own license](https://www.npmjs.com/package/rustine).
