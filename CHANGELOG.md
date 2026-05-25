# Changelog

All notable changes to the Reel GitHub Action are documented here. CLI changes live in [`getreeldev/reel-cli`](https://github.com/getreeldev/reel-cli/blob/main/CHANGELOG.md); Helm chart changes live in [`getreeldev/helm`](https://github.com/getreeldev/helm/blob/main/CHANGELOG.md).

## v1.7.2

No Action-specific changes. Released alongside Reel CLI v1.7.2, which closes a host-file overwrite vulnerability in tar extraction reachable via `reel export malware --image` and `reel export cbom --image` on untrusted images, extends the v1.7.1 pipe-keepalive shim to four more export commands, and consolidates tar extraction to a single hardened implementation. The Action writes outputs to files via `-o`, so the pipe-keepalive change is transparent. The security fix applies to any Action invocation that scans untrusted images — upgrading is recommended. See the [CLI changelog](https://github.com/getreeldev/reel-cli/blob/main/CHANGELOG.md) for details. No `action.yml` changes required.

## v1.7.1

No Action-specific changes. Released alongside Reel CLI v1.7.1, which adds a stdout pipe-keepalive shim so `reel export sbom|sarif|cbom` survives downstream stdin-readiness timeouts (e.g. piping to `claude`). The Action writes to files via `-o`, not stdout, so this change is transparent. See the [CLI changelog](https://github.com/getreeldev/reel-cli/blob/main/CHANGELOG.md) for details. No `action.yml` changes required.

## v1.7.0

No Action-specific changes. Released alongside Reel CLI v1.7.0, which overhauls CRIU detection and installation (segfault fixes, marker-file capability contract, source-aware status reporting). All changes are agent/runtime-side; the standalone Action's image-scan path is unaffected. See the [CLI changelog](https://github.com/getreeldev/reel-cli/blob/main/CHANGELOG.md) for details. No `action.yml` changes required.

## v1.6.0

No Action-specific changes. Released alongside Reel CLI v1.6.0, which adds the `--socket` flag, actionable runtime-detection errors, and VEX annotation in the scheduler — and tightens the scheduler verb namespace (`export` = local, `upload` = S3). See the [CLI changelog](https://github.com/getreeldev/reel-cli/blob/main/CHANGELOG.md) for migration details if your `reel.io/schedule` annotations use `export X --s3-bucket`. No `action.yml` changes required.

## v1.5.3

No Action-specific changes. Released alongside Reel CLI v1.5.3, which fixes a release-pipeline issue where the CLI binary inside the agent container image reported `-rc.N` instead of the GA version — see the [CLI changelog](https://github.com/getreeldev/reel-cli/blob/main/CHANGELOG.md) for details. No `action.yml` changes required; tarball downloads consumed by the Action are unchanged.

## v1.5.2

No Action-specific changes. Released alongside Reel CLI v1.5.2, which ships three CLI quality-of-life fixes (`schedule` compact table, `license` duration humanization, `status` capabilities → features rewording). See the [CLI changelog](https://github.com/getreeldev/reel-cli/blob/main/CHANGELOG.md) for details. No `action.yml` changes required.

## v1.5.1

No Action-specific changes. Released alongside Reel CLI v1.5.1, which fixes binary-blob uploads (layer / memory / checkpoint / frame) honoring the `aws-credentials` K8s secret on non-EC2 clusters — affects scheduled S3 uploads from the agent, not the standalone Action. See the [CLI changelog](https://github.com/getreeldev/reel-cli/blob/main/CHANGELOG.md) for details. No `action.yml` changes required.

## v1.5.0

No Action-specific changes. Released alongside Reel CLI v1.5.0, which adds VEX support via `--scanners vex` on `reel export sbom`. Opt in from the Action by passing `scanners: vuln,vex`:

```yaml
- uses: getreeldev/reel-action@v1
  with:
    image: redhat/ubi9-minimal
    scan-types: sarif
    scanners: vuln,vex
```

Vendor `not_affected` CVEs land in GitHub Code Scanning as pre-dismissed findings via SARIF suppressions.

## v1.4.0

Overhauls the GitHub Action's output and closes the SARIF loop with GitHub Code Scanning.

- **Scan summary on the run page** — every run now appends a markdown summary to `$GITHUB_STEP_SUMMARY` covering SBOM, SARIF, CBOM, and malware scans. Shows per-scan findings, status (`[PASS]` / `[FAIL]` / `[WARN]` / `[INFO]`), and the top vulnerabilities by severity when findings exist. Runs on every invocation — no more bare green-check pass with zero visibility, no more grep-the-artifact JSON on fail.
- **SARIF → Code Scanning** — when `scan-types` includes `sarif`, findings now upload to the repo's Security tab via `github/codeql-action/upload-sarif@v3`. Enables dismissal (persists across runs via GCS fingerprinting), PR-level gating through the "Code Scanning results" status check (configurable severity threshold in repo settings + branch protection), and cross-scanner visibility alongside CodeQL. **Requires callers to grant `security-events: write` permission** on the job.
- **SBOM severity now reflects max across all rating sources** — CycloneDX emits per-source severity ratings (nvd, alma, redhat, ubuntu, …). Summary previously implicitly picked the first source, which was non-deterministic and under-reported (e.g. a CVE rated `medium` by one source and `critical` by another showed as `medium`). Now shows the max severity across sources.
- **`scanners` input default flipped** from `''` to `'vuln'`. Without an explicit scanner, reel's SBOM emits zero vulns regardless of image content; the previous default silently produced ungated pipelines. Consumers who set `scanners:` explicitly are unaffected.
- **New `cbom-file` output** for symmetry with `sbom-file` / `sarif-file` / `malware-file`.
- **`vuln-count` and `malware-count` outputs now emit on every run** regardless of `fail-on-findings`. Previously they were only emitted when `fail-on-findings: true`.

### Compatibility

- Callers using `scan-types: sarif` must add `security-events: write` to their job permissions. Without it, the new upload step fails.
- Callers relying on `scanners: ''` (explicit empty) to skip vuln scanning should now pass the scan types they want (e.g. `scanners: secret,license`); the default behavior is now to scan vulns.

## v1.3.0

Two new pass-through inputs surface CLI flags that landed on the reel side, plus a dead-script cleanup.

- **`local` input** — pass-through for `reel export --local`. When `true`, image scans are restricted to the runner's container daemon and fail fast if the image isn't present. Default is `false`. With no flag set, reel CLI v1.3.0's local-first-with-fallback default already finds images built into the runner's daemon, so CI workflows that build-then-scan no longer need any special configuration.
- **`ignore-unfixed` input** — pass-through for `reel export --ignore-unfixed`. Applied to `sbom` and `sarif` scan types only (cbom and malware don't carry a "fixed" concept). When `true`, unfixed HIGH/CRITICAL CVEs don't trip the `fail-on-findings` gate — useful for release pipelines that want to block on fixable issues without breaking on upstream patch lag.

### Compatibility

- Requires reel CLI v1.3.0 or later in the runner (delivered via `reel-version: latest` by default). Earlier CLIs on `cbom` / `malware` don't recognize `--local`; setting `local: true` against a pinned older version will error.

### Removed

- `test-action.sh` deleted. The jq-based gate tests duplicated trivial expressions and the reel-backed tests depended on drifting upstream CVE state; nothing in CI invoked the script.

## v1.2.0

- **`fail-on-findings` input** — fail the workflow step if any vulnerabilities or malware survive the scan filters. Scans always complete and produce artifacts; the gate evaluates after.
- **`scanners` input** — passthrough to Trivy `--scanners` (vuln, secret, license, config, all).
- **`severity` input** — passthrough to Trivy `--severity` (LOW, MEDIUM, HIGH, CRITICAL).
- **Structured outputs** — `sbom-file`, `sarif-file`, `malware-file` (paths), `vuln-count`, `malware-count` (counts).

## v1.1.0 and earlier

Initial GitHub Action releases.
