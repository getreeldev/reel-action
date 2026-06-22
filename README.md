# Reel GitHub Action

Container security scanning for GitHub Actions — SBOM, vulnerability scanning (SARIF → GitHub Code Scanning), VEX annotation, cryptographic bill of materials, and malware detection. Powered by the [Reel CLI](https://github.com/getreeldev/reel-cli).

Runs on `amd64` and `arm64` Linux runners — the runner architecture is auto-detected.

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

## VEX annotation

Add `vex` to `scanners` to annotate CVEs with vendor VEX statements from [vex.getreel.dev](https://vex.getreel.dev). Vendor `not_affected` CVEs are pre-dismissed in GitHub Code Scanning via SARIF suppressions.

```yaml
- uses: getreeldev/reel-action@v1
  with:
    image: redhat/ubi9-minimal
    scan-types: sarif
    scanners: vuln,vex
```

Works with `sbom` (CycloneDX output annotated in-place) and `sarif` (vendor dismissals injected as `results[].suppressions[]`).

## Inputs

| Input | Description | Default |
|---|---|---|
| `image` | Container image to scan | _required_ |
| `scan-types` | Comma-separated: `sbom`, `cbom`, `sarif`, `malware` | `sbom` |
| `format` | SBOM format: `cyclonedx`, `spdx`, `spdx-json` | `cyclonedx` |
| `scanners` | Trivy scanners (`vuln,secret,license,config,all`) or reel pseudo-scanner (`vex`) | `vuln` |
| `severity` | Severity filter: `LOW,MEDIUM,HIGH,CRITICAL` | _all_ |
| `fail-on-findings` | Fail the workflow step on findings | `false` |
| `ignore-unfixed` | Drop unfixed vulns from the fail gate (sbom + sarif only) | `false` |
| `local` | Restrict to runner's local container daemon | `false` |
| `reel-version` | Reel CLI version (tag, e.g. `v1.7.0`) | `latest` |
| `output-dir` | Directory for scan outputs | `reel-results` |

## Outputs

| Output | Description |
|---|---|
| `sbom-file` / `sarif-file` / `cbom-file` / `malware-file` | Paths to generated artifacts (when produced) |
| `vuln-count` | Vulnerability count from SBOM, post-filter |
| `malware-count` | Infected file count |

## Gating

Two complementary gates that can be enabled together:

- **Workflow step** — set `fail-on-findings: true` to fail the run when SBOM vulns or malware hits survive filters. Fast feedback, not dismissal-aware.
- **PR merge** — set up the "Code Scanning results" status check in branch protection. Gates on GCS alerts above the repo's severity threshold; persists dismissals across runs.

Each run also appends a markdown summary to the Actions run page (`$GITHUB_STEP_SUMMARY`) with per-scan findings counts and top vulnerabilities.

## Related

- **Reel CLI binaries & changelog**: [`getreeldev/reel-cli`](https://github.com/getreeldev/reel-cli)
- **Helm chart**: [`getreeldev/helm`](https://github.com/getreeldev/helm)
- **Docs**: [getreel.dev/docs](https://getreel.dev/docs)

## License

Proprietary. See [LICENSE](LICENSE).
