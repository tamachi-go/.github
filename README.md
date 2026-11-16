## .github

orgsの情報をまとめるものです

## セットアップ

コミット時に [lefthook](https://github.com/evilmartians/lefthook) でステージしたファイルに cSpell をかけます (CI の `cSpell` ワークフローと同じチェック)。lefthook と Node.js (cSpell を `npx` で動かす用) のバージョンは [mise](https://mise.jdx.dev) で `mise.toml` に固定しています。Docker は不要です。

clone 後に一度だけ次を実行してください。

```sh
mise install                   # lefthook・Node.js を入れる
mise exec -- lefthook install  # Git フックを設定
```

## Renovate の共通設定

[renovate-config.json](./renovate-config.json) は org 内のリポジトリで共有する Renovate のプリセットです。各リポジトリの `renovate.json` から次のように読み込みます。

```json
{
  "extends": ["github>tamachi-go/.github:renovate-config"]
}
```

GitHub Actions (`uses:` と `.github/workflows/actions.lock`) の更新は各リポジトリの Dependabot が担当し、それ以外を Renovate が担当します。このリポジトリ自体も `renovate.json` でこのプリセットを読み込み、`mise.toml` のツールを Renovate で更新しています。

## cSpell のワークフローテンプレート

[workflow-templates/cspell.yml](./workflow-templates/cspell.yml) は org のワークフローテンプレートです。各リポジトリの Actions タブ → New workflow から「cSpell」を選ぶと追加できます。あわせて次の 2 ファイルをリポジトリ直下に置いてください。

`cspell.config.yaml`

```yaml
$schema: https://raw.githubusercontent.com/streetsidesoftware/cspell/main/cspell.schema.json
version: '0.2'
dictionaryDefinitions:
    - name: project-words
      path: ./project-words.txt
dictionaries:
    - project-words
ignorePaths:
    - 'node_modules'
```

`project-words.txt` (辞書に無い単語を 1 行 1 語で追加)

```text
tamachi
```

ワークフローは `actions.lock` で固定しているので、追加後に `gh actions-lock` を実行して `.github/workflows/actions.lock` もコミットしてください。

Dependabot が更新するのは `.github/workflows/` 配下だけです。`.github/workflows/cspell.yml` の Actions が更新されたら、`workflow-templates/cspell.yml` も同じバージョンに手動で揃えてください。

## 新しいリポジトリを作るときのチェックリスト

リポジトリの設定は org から引き継がれないので、作成後に既存リポジトリ (例: [slide](https://github.com/tamachi-go/slide)) に揃えます。

- [ ] Settings → General → Pull Requests
  - Allow squash merging だけ ON (Default commit message は「Pull request title and commit details」)、merge commit と rebase merging は OFF
  - Always suggest updating pull request branches を ON
  - Automatically delete head branches を ON
- [ ] `main` のブランチ保護で必須チェックに `cspell` (ビルドなどのジョブがあればそれも。例: slide は `build`) を追加し、Require branches to be up to date を ON。必須チェックの候補はワークフローが一度実行されるまで UI に出ないので、下の cSpell を追加して CI を回してから設定します。Free プランなので private リポジトリではブランチ保護を使えません (lefthook と CI で守り、CI が通ってからマージします)
- [ ] Code security
  - public: org の構成「tamachi-go default」が新規 public リポジトリに自動で適用されます (Dependabot alerts・secret scanning・push protection)
  - private: GitHub Advanced Security ベースの org 構成を使えないため、Settings → Code security で Dependabot alerts を手動で ON にします。Dependabot security updates は OFF のままにします (修正 PR は Renovate が出すため)
- [ ] [cSpell のワークフローテンプレート](#cspell-のワークフローテンプレート) から `.github/workflows/cspell.yml` を追加し、`cspell.config.yaml`・`project-words.txt` を置く。`gh actions-lock` (初回のみ `gh extension install github/gh-actions-lock`) で `.github/workflows/actions.lock` を生成してコミット (lock は実行時に強制されるので、無いとワークフローが失敗します)
- [ ] `.github/dependabot.yml` で GitHub Actions だけを更新対象にする

  ```yaml
  version: 2
  updates:
    - package-ecosystem: github-actions
      directory: /
      schedule:
        interval: weekly
      cooldown:
        default-days: 7
  ```

- [ ] 依存がある場合は `renovate.json` で [Renovate の共通設定](#renovate-の共通設定) を読み込む。Renovate アプリは org 全体に入っているので onboarding PR が自動で作られます (先に自前の `renovate.json` をマージするか、PR を閉じる)
- [ ] `mise.toml` と `lefthook.yml` を置き、コミット時に cSpell をかける ([セットアップ](#セットアップ) とこのリポジトリのファイルをそのまま流用できます)
- [ ] public の場合は `LICENSE` (MIT) を置き、README のライセンス欄で第三者の素材 (ロゴ・画像など) を対象外と明記する ([slide](https://github.com/tamachi-go/slide#ライセンス) や下記を参照)
- [ ] (任意) About の Description・Website を設定する

## ライセンス

[MIT License](./LICENSE)

`profile/square.png` は MIT License の対象外です（ライセンスは [tamachi-go/assets](https://github.com/tamachi-go/assets#ライセンス) を参照）
