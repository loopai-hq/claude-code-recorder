# Changelog

All notable changes to this project are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project
uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Each
released section's heading is `## [X.Y.Z] - YYYY-MM-DD` exactly, because the
release workflow extracts it for the release notes; the date is the day the
tag is cut and is set in the same change as the tag.

## [0.2.0] - 2026-10-22

The project is renamed from loop-sessions to claude-code-recorder. The
repository is `loopai-hq/claude-code-recorder` (GitHub redirects the old
URLs), and the product, the binary, the module, the image, the installer
and the documentation follow. It remains unaffiliated with and not
endorsed by Anthropic.

### Changed
- **Names.** The Go module is `github.com/loopai-hq/claude-code-recorder`;
  the client is `cmd/claude-code-recorder` and the server
  `server/cmd/claude-code-recorder-server`. Release assets are
  `claude-code-recorder_{darwin,linux}_{amd64,arm64}`, attested under
  those names; `SHA256SUMS`, its cosign bundle, `latest.json`, the two
  scripts and `THIRD_PARTY_NOTICES.md` keep their names. The server image
  is `ghcr.io/loopai-hq/claude-code-recorder-server`.
- **Environment variables.** Every `LOOP_SESSIONS_*` variable is
  `CLAUDE_CODE_RECORDER_*`: the installer's `BASE_URL`, `VERSION`,
  `BIN_DIR`, `NO_ENROLL` and `ENDPOINT`; the client's `HOME`, `NO_UPGRADE`
  and `SLACK`; the server's `SLACK_SIGNING_SECRET`; the test suite's
  `TEST_DSN` and `BIG_FIXTURE*`. The demo server's `LOOP_WEB_DEMO` is
  `CLAUDE_CODE_RECORDER_WEB_DEMO`.
- **On the laptop.** The client's state directory is
  `~/.claude-code-recorder` (was `~/.loop/sessions`); the hook entries in
  `~/.claude/settings.json` carry `#claude-code-recorder`; the User-Agent
  is `claude-code-recorder/<build>`; the scrubber's kind for this
  service's own tokens is `claude_code_recorder_token`.
- **The export and the examples.** The export job's default database name
  and dataset are `claude_code_recorder` and `claude_code_recorder_raw`
  (`DATABASE_NAME` and `EXPORT_DATASET` override them), and its BigQuery
  job ids start with `claude_code_recorder_export_`. In the GCP example
  the Cloud Run service, the export job and its schedule, the service
  accounts, the secrets, the Artifact Registry repository, the release
  bucket, the log-based metrics (`claude_code_recorder/*`) and alert
  policies, the datasets and the policy-tag taxonomy are renamed, and in
  the compose example the project and the database. Table and column
  names are unchanged.
- The identifier gate's allowlist names the new repository and no longer
  carves out `loop-plugins`, which is private.

### Migration
- **Laptops.** An installed `loop-sessions` agent does not upgrade itself
  across the rename: it looks for `loop-sessions_<os>_<arch>` in the
  manifest, finds no such asset, and stays on its build (its health
  report says so). On each machine, run the installer again (the
  one-liner in the README, or the server's `/install` page), then
  `claude-code-recorder install`. The install moves `~/.loop/sessions` to
  `~/.claude-code-recorder` whole, so the configuration, the device token
  and any queued events come along and nobody signs in again, and it
  replaces the `#loop-sessions` hook entries with its own. Daemons of the
  old binary that are still running end with their sessions; then delete
  `~/.local/bin/loop-sessions`. Running `loop-sessions uninstall` with
  the old binary first also works: it keeps the old directory, which the
  new install then moves. `CLAUDE_CODE_RECORDER_HOME` set to somewhere
  else skips the move. `claude-code-recorder uninstall` and
  `uninstall.sh` remove hook entries under either marker and name the old
  directory when it is still there.
- **Servers.** Pull the new image. Rename
  `LOOP_SESSIONS_SLACK_SIGNING_SECRET` to
  `CLAUDE_CODE_RECORDER_SLACK_SIGNING_SECRET` where it is set; the server
  starts without it, with the Slack interactivity route absent and no
  Stop buttons, which is how a missed rename shows. The Postgres schema is
  unchanged. A server that is a release host publishes the new assets to
  its channel; its `/dl` allowlist names only those.
- **The export.** An existing deployment keeps writing where it did by
  setting `DATABASE_NAME` and `EXPORT_DATASET` to the values it was
  provisioned with, as `examples/deploy-gcp/analytics/job.yaml` sets them
  for a new one. The example's other resource names are a template; a
  deployment provisioned under the old names keeps them.
- **What keeps the old name, and why:**
  - `internal/hooks` treats `#loop-sessions` entries as its own, so one
    `install` replaces them and one `uninstall` removes them.
  - `internal/config.AdoptLegacyRoot`, `install/install.sh` and
    `install/uninstall.sh` know `~/.loop/sessions`: the first moves it,
    the scripts name it.
  - `server/ingest.AgentBuild` and the example's two User-Agent log
    metrics (`ingest_5xx`, `health_posts`) accept `loop-sessions/<build>`,
    so the fleet page and the metrics keep naming the build of a laptop
    that has not reinstalled.
  - `install/uninstall.sh` looks for a launch agent under the old name as
    well as the new.
  - The 0.1.0 section below describes that release as it shipped.
  - The device and source token prefixes, `lsd_` and `lss_`, are the
    credentials in the field and are unchanged.

## [0.1.0] - 2026-10-06

The first public release: the client agent, the server, the dashboard and
the example deployments, with releases people can download and verify. The
repository is `loopai-hq/loop-sessions`, matching the binary and the
product name, and the Go module path follows
(`github.com/loopai-hq/loop-sessions`); GitHub redirects the old
`agent-sessions` URLs once the rename is done (the first item of the
pre-tag checklist in `docs/MAINTAINING.md`).

### Added
- GitHub Releases for every `vX.Y.Z` tag, with the four agent binaries
  (`loop-sessions_{darwin,linux}_{amd64,arm64}`), `SHA256SUMS` and its
  keyless cosign signature `SHA256SUMS.sigstore.json`, `latest.json`,
  `install.sh` and `uninstall.sh` with their digests, and
  `THIRD_PARTY_NOTICES.md`. Every binary named in `SHA256SUMS`, both
  scripts and the image digest are attested with build provenance
  (`gh attestation verify <asset> --repo loopai-hq/loop-sessions`);
  `SHA256SUMS` is signed with cosign; the remaining assets are covered by
  the checksums and the signature, not by an attestation of their own.
- The server image `ghcr.io/loopai-hq/loop-sessions-server`, tagged
  `vX.Y.Z`, `sha-<full commit>` and `latest`, with OCI labels, attested;
  the release workflow verifies the attestation before it publishes.
- The installer works from GitHub Releases without a channel host
  (`LOOP_SESSIONS_BASE_URL=.../releases/latest LOOP_SESSIONS_VERSION=download`).
- `pause --for <duration>`: capture and delivery pause for the period and
  resume on their own; `resume` ends it early; `status` and `doctor` show
  the deadline, and the health report stops reporting paused once it has
  passed (no new wire field). New config field `paused_until`.
- `install` and `status` print who can read captured sessions: the owner
  and the server's admins, who also see every session's first prompt and
  metadata in the list; a colleague only through a share link the owner or
  an admin creates; a full read of someone else's session is written to the
  access log; retention and any export are the operator's decision.
- `mirror` is listed in `loop-sessions help`.
- CI: golangci-lint v2 with a curated configuration (`.golangci.yml`,
  depguard holding the client to the standard library), a macOS test leg,
  the integration suite on Postgres 15 and 17, govulncheck, gitleaks,
  dependency review, a weekly schedule, and OpenSSF Scorecard.
- Reproducible builds: `BUILD_DATE` defaults to the commit time
  (`SOURCE_DATE_EPOCH` honoured), CI builds every commit twice and diffs
  `SHA256SUMS`.
- `make notices` and `THIRD_PARTY_NOTICES.md`, shipped in releases and in
  the image; `make screenshots` and the four dashboard screenshots in
  `docs/images/`.
- `examples/docker-compose/`: Postgres 16 plus the server in one command.
- Documentation: `docs/ARCHITECTURE.md`, `docs/OPERATIONS.md`,
  `docs/DATA-PROTECTION.md`, `docs/THREAT-MODEL.md`, `docs/MAINTAINING.md`,
  `SUPPORT.md`, `GOVERNANCE.md`, `MAINTAINERS.md`, `ROADMAP.md`,
  `AGENTS.md` and `CLAUDE.md`, YAML issue forms, and the AI-assisted
  contribution policy in `CONTRIBUTING.md`.
- `.editorconfig`, `.gitattributes`, `.github/release.yml` categories for
  generated release notes.

### Changed
- **The server Dockerfile moved from `examples/deploy-gcp/Dockerfile` to the
  repository root** (with a root `.dockerignore`). Cloud Build triggers or
  scripts that pass `-f examples/deploy-gcp/Dockerfile` must drop the flag;
  the build context stays the repository root. New build arg `REVISION`
  (the full commit) for the OCI label; `BUILD_DATE` should be the commit
  time.
- Dependencies: pgx v5.11.0, goldmark v1.8.6, `golang.org/x/sync` v0.23.0,
  `golang.org/x/text` v0.42.0; `go 1.26.0` with `toolchain go1.26.8`; the
  Dockerfile base is `golang:1.26.8-bookworm`. These supersede the four
  Dependabot pull requests open since the org move. Dependabot now groups
  Go minor and patch updates, groups Actions updates, and tracks the
  Dockerfile's base image.
- The README is rewritten in the exemplar shape: product proof first, a
  five-minute path that needs no Firebase, a supported-harnesses table, a
  data-inventory table, and honest platform claims (macOS and Linux only;
  Codex is imported and repaired from disk, not captured live). The design
  rationale moved to `docs/ARCHITECTURE.md`.
- `server/CONTRACT-*.md` moved to `docs/design/` as dated historical notes.
- The `discover` hint names the real mechanism for unfound session
  directories (`"roots"` in `config.json`) instead of a `--set` flag that
  did not exist.
- Every exported identifier has a doc comment; builtin identifiers are no
  longer shadowed; `%w` wrapping and `errors.Is` where the auto-fixer could
  prove it; one fewer trailing newline in an enrolment error message.
- `discovery.json` in the client's state directory is written `0600`
  (was `0644`), matching every other file there.
- `SECURITY.md` states a response SLA and a supported-versions table;
  issue templates are YAML forms; the pull request template asks for an AI
  assistance line.

### Fixed
- `pause --for` was accepted and silently ignored, leaving a machine paused
  until someone ran `resume`.
- `loop-sessions help` did not list `mirror`.
- The README claimed Windows support and live Codex capture; neither was
  true.
- `reflect.Ptr` in a test fake (deprecated) and the pgx `TypeMap` the new
  driver version requires of row fakes.
- `docs/RELEASE.md` claimed reproducible builds while `BUILD_DATE` was the
  wall clock; the claim is now true and proven in CI.

### Removed
- `LaunchAgentLabel`, an unused constant carrying the previous organisation
  name.
- `examples/deploy-gcp/Dockerfile` and its `.dockerignore` (see Changed).
- The Markdown issue templates, replaced by YAML forms.

### Security
- `gitleaks` runs in CI over the tree and the history with the repository
  configuration; the three known fixture false positives are allowlisted by
  path and exact string, never by widening the paths.
- Every GitHub Action is pinned to a full commit SHA; `persist-credentials:
  false` on every checkout; least-privilege `GITHUB_TOKEN` with job-level
  elevation only where the release publishes.
- `docs/THREAT-MODEL.md` documents the trust boundaries, what a stolen
  device token can do, the third-party script on the sign-in page, and the
  read-then-run form of the installer.

[0.2.0]: https://github.com/loopai-hq/claude-code-recorder/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/loopai-hq/claude-code-recorder/releases/tag/v0.1.0
