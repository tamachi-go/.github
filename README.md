## .github

orgsの情報をまとめるものです

## Renovate の共通設定

[renovate-config.json](./renovate-config.json) は org 内のリポジトリで共有する Renovate のプリセットです。各リポジトリの `renovate.json` から次のように読み込みます。

```json
{
  "extends": ["github>tamachi-go/.github:renovate-config"]
}
```

GitHub Actions (`uses:` と `.github/workflows/actions.lock`) の更新は各リポジトリの Dependabot が担当し、それ以外を Renovate が担当します。このリポジトリ自体は Actions しか依存がないので Renovate は使っていません。
