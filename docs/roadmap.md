# Roadmap

Repolica is intentionally starting with a narrow disaster-recovery use case before expanding into a general multi-forge replication control plane.

## v0.1 — GitHub to GitLab replication

Goal: provide a trustworthy, self-hosted one-way replication loop for individual users and small installations.

### Configuration

- [ ] Parse and validate `repolica.yaml`.
- [ ] Read credentials from environment variables / secret mounts.
- [ ] Support repository include/exclude filters.

### GitHub source adapter

- [ ] Authenticate to GitHub.
- [ ] Discover repositories for a user or organization.
- [ ] Read clone URLs and repository visibility.
- [ ] Handle pagination and private repositories.

### GitLab target adapter

- [ ] Authenticate to GitLab Self-Managed.
- [ ] Detect an existing target project.
- [ ] Create a missing target project.
- [ ] Map namespace and visibility safely.

### Replication engine

- [ ] Create or reuse a local bare mirror cache.
- [ ] Fetch and prune source refs.
- [ ] Push the intended ref set to the target.
- [ ] Isolate failures per repository.
- [ ] Prevent secrets from appearing in logs.

### Verification

- [ ] Enumerate source refs.
- [ ] Enumerate target refs.
- [ ] Compare expected ref tips.
- [ ] Classify `healthy`, `behind`, `diverged`, and `error` states.

### CLI

- [ ] `repolica sync`
- [ ] `repolica status`
- [ ] `repolica check`
- [ ] Human-readable output.
- [ ] Machine-readable JSON output.
- [ ] Meaningful exit codes for automation.

### Distribution

- [ ] Linux binary release.
- [ ] macOS binary release.
- [ ] Windows binary release where practical.
- [ ] Container image.
- [ ] Docker Compose example.

## v0.2 — Operational hardening

Goal: make Repolica boring to operate continuously.

- [ ] Built-in scheduler / daemon mode.
- [ ] Structured logs.
- [ ] Retry policy with bounded exponential backoff.
- [ ] Sync locks to prevent overlapping runs.
- [ ] Health endpoint.
- [ ] Prometheus/OpenTelemetry metrics.
- [ ] Dry-run planning.
- [ ] More complete filtering policies.
- [ ] Repository rename/deletion policy.

## v0.3 — Independent snapshots

Goal: make destructive changes recoverable instead of merely replicating them.

- [ ] Define snapshot format and restore contract.
- [ ] S3-compatible object storage target.
- [ ] Cloudflare R2 compatibility.
- [ ] Retention policies.
- [ ] Optional immutability/object-lock integration where supported.
- [ ] `repolica snapshot` and `repolica restore` workflows.

## v0.4 — More forges

- [ ] GitLab source adapter.
- [ ] Forgejo adapter.
- [ ] Gitea adapter.
- [ ] Multiple targets per replication set.
- [ ] Provider capability matrix.

## Later / research

These are intentionally not commitments yet:

- Provider metadata export for issues, pull requests / merge requests, releases, and settings.
- Webhook-triggered synchronization.
- Read-only dashboard.
- Failover runbooks and assisted promotion of a replica to primary.
- Encrypted off-site snapshot manifests.
- Repository integrity auditing beyond ref-tip comparison.

## Non-goals

Unless the architecture changes deliberately, Repolica is not intended to become:

- A bidirectional collaborative sync engine.
- A replacement Git forge.
- A secrets replication system.
- A transparent active-active Git cluster across unrelated providers.
