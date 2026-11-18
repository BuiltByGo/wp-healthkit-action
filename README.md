# WP HealthKit Audit Action

Run a full WP HealthKit security and quality audit on a WordPress plugin as part of your
GitHub Actions workflow. The action packages the plugin, runs the complete audit pipeline,
writes SARIF for GitHub code scanning, and fails the job at a severity threshold you choose.

Published from this directory to the public
[`BuiltByGo/wp-healthkit-action`](https://github.com/BuiltByGo/wp-healthkit-action) repository.
The monorepo is private; use the public repository (or a pinned SHA) in workflows.

## Usage

```yaml
name: Plugin Audit
on:
  pull_request:
    paths:
      - "**/*.php"
      - "**/composer.json"
      - "**/composer.lock"
      - "**/package.json"
      - "**/package-lock.json"

permissions:
  contents: read
  pull-requests: write
  security-events: write

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: BuiltByGo/wp-healthkit-action@v1
        id: audit
        with:
          api-key: ${{ secrets.WP_HEALTHKIT_API_KEY }}
          fail-on: HIGH

      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: wphk-audit.sarif
          category: wp-healthkit
```

## Requirements

- A **Platform or Enterprise** WP HealthKit plan. Create the key in
  Settings → API keys and store it as a repository secret.
- A Linux runner with `curl`, `jq` and `zip` on `PATH`. `ubuntu-latest` has all three; the
  action fails fast with a clear message on images that do not.
- Node.js is not required — the action is pure shell.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `api-key` | required | WP HealthKit API key (Platform or Enterprise). |
| `plugin-path` | `.` | Directory of the plugin. Point at a subfolder in a monorepo. |
| `fail-on` | `HIGH` | `CRITICAL`, `HIGH`, `MEDIUM`, `LOW` or `none`. |
| `api-url` | `https://wphealthkit.com` | Override for self-hosted or staging. |
| `sarif` | `true` | Write `wphk-audit.sarif` for code scanning. |

## Outputs

| Output | Description |
| --- | --- |
| `report-url` | Link to the full report. |
| `overall-risk` | `CRITICAL`, `HIGH`, `MEDIUM`, `LOW` or `INFO`. |
| `findings-count` | Total findings. |
| `critical-count` | Critical findings. |
| `high-count` | High findings. |

## Security notes

- The API key is passed through the environment, never interpolated into the command line,
  so it does not appear in the rendered run script.
- The packaged ZIP excludes `.git`, `node_modules`, `vendor`, `.env*`, private keys,
  `.htpasswd`/`.htaccess` and `wp-config.php`.
- Pin to a commit SHA in production if you want reproducible runs; `v1` floats to the
  latest compatible release.

## Troubleshooting

- **401** — the secret is missing, revoked, or belongs to another account.
- **403** — the key's plan does not include Action audits (Platform or Enterprise required).
- **429** — the monthly audit quota is exhausted. Add `paths:` filters or upgrade.
- **Check never blocks a merge** — Actions jobs block only when the workflow/job is marked
  required in branch protection.

Full docs: <https://wphealthkit.com/docs/github>

## License

MIT. See `LICENSE`.
