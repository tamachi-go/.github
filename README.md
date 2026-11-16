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

## ライセンス

[MIT License](./LICENSE)

`profile/square.png` は MIT License の対象外です（ライセンスは [tamachi-go/assets](https://github.com/tamachi-go/assets#ライセンス) を参照）
