# Architecture

## Purpose

Repolica continuously replicates Git repositories from a designated source forge to one or more target forges and verifies that the resulting Git refs match the intended source state.

The project treats replication and backup as separate concerns:

- **Replication** keeps another forge ready with the current repository state.
- **Backup** preserves recoverable historical states outside the replication path.

## Core invariants

### 1. Single writer

A Repolica replication set has one authoritative source at a time.

```text
source -> Repolica -> target(s)
```

Targets are read-only from Repolica's perspective during normal operation. Bidirectional synchronization is intentionally excluded from the initial design because concurrent writes make conflict semantics substantially harder to reason about.

### 2. Provider-neutral core

Provider-specific API behavior must not leak into the synchronization engine.

```text
                 +------------------+
                 |   Sync Engine    |
                 +---------+--------+
                           |
                provider interfaces
                           |
             +-------------+-------------+
             |                           |
        GitHub adapter              GitLab adapter
```

Adapters are responsible for operations such as repository discovery, target project creation, authentication, visibility mapping, and provider metadata needed for Git transport.

### 3. Git is the replication boundary

The first replication contract covers Git repository data: refs and the objects reachable from those refs.

Provider-native metadata is not part of the v0.1 replication contract. That includes issues, pull requests / merge requests, discussions, CI definitions outside the repository, secrets, branch protection settings, and project-level configuration.

### 4. Verification is part of synchronization

A transport command returning success does not by itself mean the replica is healthy.

After synchronization, Repolica should compare the source and target ref sets and classify the repository state, for example:

- `healthy` — expected refs match.
- `behind` — one or more target refs are older or missing.
- `diverged` — a target ref contains a different tip than the source.
- `error` — discovery, provisioning, transport, or verification failed.

### 5. Replication is not backup

A destructive source change can be faithfully propagated to every replica. Therefore snapshots must be stored independently and must not be implemented merely as another `--mirror` target.

Snapshot support is planned after the core replication loop is stable.

## v0.1 data flow

```text
+------------------+
| repolica.yaml    |
+--------+---------+
         |
         v
+------------------+       GitHub API
| Config + Planner |<--------------------+
+--------+---------+                     |
         |                               |
         v                               |
+------------------+             +-------+-------+
| Repository Plan  |             | GitHub source |
+--------+---------+             +-------+-------+
         |                               |
         | ensure target                 | Git transport
         v                               |
+------------------+                     |
| GitLab adapter   |                     |
+--------+---------+                     |
         |                               |
         v                               v
+------------------------------------------------+
|              Mirror / Sync Engine              |
+------------------------+-----------------------+
                         |
                         v
                +-----------------+
                | Ref Verification|
                +--------+--------+
                         |
                         v
                +-----------------+
                | Status / Output |
                +-----------------+
```

## Proposed Go package layout

```text
cmd/
  repolica/
internal/
  config/
  provider/
    github/
    gitlab/
  planner/
  mirror/
  verify/
  status/
```

The first implementation should prefer the system `git` executable over embedding a full Git implementation. This keeps behavior close to normal Git tooling and reduces the amount of protocol behavior Repolica must own. The abstraction should nevertheless isolate command execution so it can be tested deterministically.

## Configuration model

The configuration describes intent, not provider-specific execution steps.

```yaml
source:
  provider: github
  owner: hamakyo

targets:
  - provider: gitlab
    url: https://gitlab.example.com
    namespace: hamakyo

sync:
  interval: 10m
```

Authentication material should be supplied through environment variables or secret mounts rather than committed configuration files.

## Failure behavior

Repolica must fail per repository where possible instead of aborting an entire run after one repository fails.

A synchronization run should therefore produce an aggregate result such as:

```text
12 repositories checked
10 healthy
 1 behind
 1 error
```

Retries should be bounded and observable. Repolica must not hide persistent failure behind infinite retries.

## Security boundaries

The service can require credentials with access to private source repositories and permission to create or write target repositories. The initial implementation should follow least privilege and avoid logging credentials or authenticated remote URLs.

Credentials must never be copied from the source provider to the target provider.

## Future extensions

Potential post-v0.1 work includes:

- GitLab as a source
- Forgejo and Gitea adapters
- Multiple replication targets
- S3-compatible immutable snapshots
- Repository inclusion/exclusion policies
- Webhook-triggered sync in addition to polling
- Prometheus/OpenTelemetry observability
- A read-only web dashboard
- Provider metadata export as a separate disaster-recovery feature

These should extend the core without weakening the single-writer and replication-versus-backup boundaries.
