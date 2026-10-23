# security-workflows

Web サイトのリポジトリで使う、週次のセキュリティ点検の共通ワークフローです。

## 点検の流れ

1. 直前 7 日間にコミットがあるかを確かめる。なければ点検しない（手動実行のときは必ず点検する）
2. OSV-Scanner でロックファイルを調べ、依存パッケージの既知の脆弱性を `osv-results.json` に書き出す
3. Claude Code が `security-audit` スキル（`skills/security-audit/`）でリポジトリ全体を点検し、重大度つきのレポートを書く
4. レポートを Issue として登録する

Claude が動くジョブには読み取り権限しか与えず、Issue を作る権限は別のジョブにだけ与えています。

## 使い方

呼び出す側のリポジトリに、次のファイルを置きます。

```yaml
# .github/workflows/security-audit.yml
name: Security Audit

on:
  schedule:
    - cron: '0 1 * * 1' # 毎週月曜 10:00（JST）
  workflow_dispatch:

permissions: {}

jobs:
  security-audit:
    uses: kikeda1102/security-workflows/.github/workflows/security-audit.yml@main
    permissions:
      contents: read
      issues: write
    # pnpm 以外のときは、ロックファイルを指定する
    # with:
    #   lockfile: package-lock.json
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

あわせて、次の準備をします。

- リポジトリの secrets に `CLAUDE_CODE_OAUTH_TOKEN` を登録する
- リポジトリ固有の前提と許容済みのリスクを `.claude/security-audit-notes.md` に書く
- Dependabot alerts と security updates を有効にする
