# Reel GitHub Action

Container security scanning for GitHub Actions — SBOM generation, vulnerability scanning (SARIF → GitHub Code Scanning), cryptographic bill of materials, and malware detection. Powered by the [Reel CLI](https://github.com/getreeldev/releases).

## Usage

```yaml
jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write   # required when scan-types includes `sarif`
    steps:
      - uses: getreeldev/reel-action@v1
        with:
          image: myapp:${{ github.sha }}
          scan-types: sbom,cbom,sarif,malware
          fail-on-findings: true

      - uses: actions/upload-artifact@v4
        with:
          name: security-reports
          path: reel-results/
```

### Required permissions

When `scan-types` includes `sarif`, the caller must grant `security-events: write` so findings upload to GitHub Code Scanning. Without it, the upload step fails.

### Two-layer gating

The action supports two complementary gates:

| Gate | Trigger | What fails | Dismissal-aware |
|---|---|---|---|
| **Workflow run** | `fail-on-findings: true` on the action | Any SBOM vulns or malware hits after filters | No |
| **PR merge** | "Code Scanning results" status check in branch protection | GCS alerts from uploaded SARIF, above the repo's configured severity threshold | Yes |

For most consumers: enable both. The workflow gate gives fast feedback on the CI run; the PR gate is triage-friendly — dismissals in the Security tab persist across runs, and the severity threshold is configurable per-repo.

### Inputs

See [`action.yml`](./action.yml) for the full list. Key inputs:

| Input | Description | Default |
|---|---|---|
| `image` | Container image to scan | _required_ |
| `scan-types` | Comma-separated: `sbom`, `cbom`, `sarif`, `malware` | `sbom` |
| `format` | SBOM format: `cyclonedx`, `spdx`, `spdx-json` | `cyclonedx` |
| `scanners` | Trivy scanners: `vuln,secret,license,config,all` | `vuln` |
| `severity` | Trivy severity filter: `LOW,MEDIUM,HIGH,CRITICAL` | `''` |
| `fail-on-findings` | Fail the workflow step on findings | `false` |
| `ignore-unfixed` | Drop unfixed vulns from the fail gate | `false` |
| `local` | Restrict to runner's local container daemon | `false` |
| `reel-version` | Reel CLI version | `latest` |
| `output-dir` | Directory for scan outputs | `reel-results` |

### Outputs

| Output | Description |
|---|---|
| `sbom-file` | Path to `sbom.json` if generated |
| `sarif-file` | Path to `results.sarif` if generated |
| `cbom-file` | Path to `cbom.json` if generated |
| `malware-file` | Path to `malware.json` if generated |
| `vuln-count` | Vulnerability count from SBOM (emitted on every run) |
| `malware-count` | Infected file count from malware scan (emitted on every run) |

### Scan summary

Each run appends a readable markdown summary to the Actions run page (`$GITHUB_STEP_SUMMARY`), covering all four scan types with per-scan findings counts and a sorted list of the top vulnerabilities when findings exist.

## Related

- **Reel CLI binaries & changelog**: [`getreeldev/releases`](https://github.com/getreeldev/releases)
- **Helm chart**: [`getreeldev/helm`](https://github.com/getreeldev/helm)
- **Full documentation**: [getreel.dev/docs](https://getreel.dev/docs)
- **GitHub Action docs**: [getreel.dev/docs/github-action](https://getreel.dev/docs/github-action)

## License

Proprietary. See [LICENSE](LICENSE) for terms.
