<h1 align="center">Repolica</h1>

<p align="center">
  <strong>Gitリポジトリのためのセルフホスト型レプリケーション／DR基盤。</strong>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/github/license/hamakyo/repolica" alt="License"></a>
  <a href="https://github.com/hamakyo/repolica/stargazers"><img src="https://img.shields.io/github/stars/hamakyo/repolica?style=flat" alt="GitHub stars"></a>
  <a href="https://github.com/hamakyo/repolica/issues"><img src="https://img.shields.io/github/issues/hamakyo/repolica" alt="GitHub issues"></a>
  <img src="https://img.shields.io/badge/status-pre--alpha-orange" alt="Status: pre-alpha">
</p>

<p align="center">
  <a href="README.md">English</a> | 日本語
</p>

Repolicaは、特定のGitフォージだけを唯一の保管先にせず、複数のプロバイダー間でGitリポジトリを継続的に複製するためのオープンソースなコントロールプレーンです。

最初の目標は意図的に小さくしています。**GitHubを正（source of truth）とし、GitLab Self-Managedをウォームスタンバイとして維持する**構成をv0.1で実現します。

```text
GitHub (Primary)
      |
      | discover + mirror
      v
   Repolica
      |
      | provision + sync + verify
      v
GitLab Self-Managed (Replica)
```

## なぜRepolica？

リポジトリのミラーは有用ですが、ミラーリングだけでは完全な災害復旧（DR）にはなりません。Repolicaは以下の原則を明示的に採用します。

- **Single Writerを基本とする** — 通常運用時の書き込み先を1つに限定し、双方向同期による競合を避けます。
- **ReplicationはBackupではない** — Replicaは現在の状態を追従し、過去状態の保全はSnapshot/Backupが担います。
- **Provider-independent core** — GitHubやGitLab固有の処理はAdapterの内側に閉じ込めます。
- **Self-host first** — 自宅サーバー、ミニPC、VPSなどで手軽に動かせることを重視します。
- **検証可能なレプリケーション** — `git push`の成功だけでは正常と判定せず、Source/Targetのrefsを比較します。

## v0.1のスコープ

最初の利用可能なリリースでは、以下のフローを実装します。

1. `repolica.yaml` を読み込む。
2. GitHubアカウントまたはOrganizationからリポジトリを検出する。
3. GitLab Self-Managed側に存在しないProjectを自動作成する。
4. Git refsをTargetへミラーする。
5. 同期後にSource/Targetのrefsを検証する。
6. CLIからリポジトリごとのレプリケーション状態を確認できるようにする。
7. 1回だけの実行、またはスケジュールによる常駐実行に対応する。

v0.1では以下を対象外とします。

- 双方向同期
- Issues / Pull Requests / Discussionsの移行
- GitHub ActionsからGitLab CIへの変換
- Secretsの移行
- DNSやRoutingを含む自動フェイルオーバー
- Repolica自体のマルチノードHA

## 予定しているCLI

```bash
repolica sync
repolica status
repolica check
```

ステータス表示例：

```text
Repolica

OK  tilelog-lens   synced     12s ago
OK  StreamPulse    synced     14s ago
OK  okf-skills     synced     17s ago
ERR lpbench        behind      2 commits

4 repositories: 3 healthy, 1 degraded
```

## 設定例

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

設定ファイルの例は [`examples/repolica.yaml`](examples/repolica.yaml) を参照してください。

## アーキテクチャ

RepolicaのCoreは、Gitプロバイダーに依存しない設計を目指します。

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

将来的にはGitLab、Forgejo、GiteaなどをSource/Targetのどちらとしても利用できるAdapterを追加する想定です。

詳細：[`docs/architecture.md`](docs/architecture.md)

## ロードマップ

[`docs/roadmap.md`](docs/roadmap.md) を参照してください。

## 現在のステータス

**Pre-alpha**。実装に入る前に、アーキテクチャとv0.1の契約を固めている段階です。

## Contributing

Repolicaは現在設計段階です。Providerごとの挙動、障害モード、検証方法、最小限のセルフホスト要件などに関するIssueを歓迎します。

## License

MIT License。詳細は [`LICENSE`](LICENSE) を参照してください。
