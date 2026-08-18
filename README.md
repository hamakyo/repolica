<h1 align="center">Repolica</h1>

<p align="center">
  <strong>Self-hosted repository replication and disaster recovery for Git.</strong>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/github/license/hamakyo/repolica" alt="License"></a>
  <a href="https://github.com/hamakyo/repolica/stargazers"><img src="https://img.shields.io/github/stars/hamakyo/repolica?style=flat" alt="GitHub stars"></a>
  <a href="https://github.com/hamakyo/repolica/issues"><img src="https://img.shields.io/github/issues/hamakyo/repolica" alt="GitHub issues"></a>
  <img src="https://img.shields.io/badge/status-pre--alpha-orange" alt="Status: pre-alpha">
</p>

<p align="center">
  English | <a href="README.ja.md">日本語</a>
</p>

Repolica is an open-source control plane for continuously replicating Git repositories across providers without making any single forge your only copy of the source.

The initial focus is deliberately small: **GitHub as the source of truth and GitLab Self-Managed as a warm standby**.

```text
GitHub (primary)
      |
      | discover + mirror
      v
   Repolica
      |
      | provision + sync + verify
      v
GitLab Self-Managed (replica)
```

## Why Repolica?

A repository mirror is useful, but mirroring alone is not a complete disaster-recovery strategy. Repolica is designed around a few explicit principles:

- **Single writer by default** — one source of truth avoids bidirectional-sync conflicts.
- **Replication is not backup** — a replica tracks the current state; snapshots preserve previous states.
- **Provider-independent core** — provider-specific behavior lives behind adapters.
- **Self-host first** — Repolica should be easy to run on a home server, mini PC, or VPS.
- **Verifiable replication** — successful pushes are not enough; source and target refs should be checked.

## v0.1 scope

The first usable release targets this flow:

1. Read `repolica.yaml`.
2. Discover repositories from a GitHub account or organization.
3. Create missing projects on GitLab Self-Managed.
4. Mirror Git refs to the target.
5. Verify source and target refs after synchronization.
6. Report per-repository health from the CLI.
7. Run once or continuously on a schedule.

Out of scope for v0.1:

- Bidirectional synchronization
- Issues / pull requests / discussions migration
- GitHub Actions to GitLab CI conversion
- Secrets migration
- Automatic failover DNS/routing
- Multi-node Repolica high availability

## Planned CLI

```bash
repolica sync
repolica status
repolica check
```

Example status output:

```text
Repolica

OK  tilelog-lens   synced     12s ago
OK  StreamPulse    synced     14s ago
OK  okf-skills     synced     17s ago
ERR lpbench        behind      2 commits

4 repositories: 3 healthy, 1 degraded
```

## Example configuration

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

repositories:
  visibility:
    - public
    - private

backup:
  enabled: false
```

See [`examples/repolica.yaml`](examples/repolica.yaml) for the evolving configuration example.

## Architecture

The core is intentionally provider-neutral.

```text
                 +----------------+
                 |    Repolica    |
                 +-------+--------+
                         |
             +-----------+-----------+
             |                       |
        Source Adapter          Target Adapter
             |                       |
          GitHub                   GitLab
                                     |
                               Self-Managed
```

Future adapters may support GitLab, Forgejo, Gitea, and other Git-compatible providers as either sources or targets.

More detail: [`docs/architecture.md`](docs/architecture.md)

## Roadmap

See [`docs/roadmap.md`](docs/roadmap.md).

## Status

**Pre-alpha.** The architecture and v0.1 contract are being defined before implementation.

## Contributing

Repolica is at the design stage. Issues describing provider behavior, failure modes, verification semantics, and minimal self-hosting requirements are especially useful.

## License

MIT. See [`LICENSE`](LICENSE).
