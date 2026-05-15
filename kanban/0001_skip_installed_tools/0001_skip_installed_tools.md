# インストール済みツールのダウンロードステップをスキップ

## 目的
インストール済みだったら必要ないステップであるため

## 要望
`Download jwt-cli`ステップと `Download oidc-jwks-converter`ステップを既にインストール済みだったらスキップするようにしてください

## 完了サマリー

**完了日時**: 2026-05-15T15:35:00+09:00

### 実施内容
- `action.yml` の `Download jwt-cli` ステップに `command -v jwt` チェックを追加
- `action.yml` の `Download oidc-jwks-converter` ステップに `command -v oidc-jwks-converter` チェックを追加
- 両ステップで、ツールが既にインストール済みの場合は実行をスキップ、なければダウンロード実行

### 修正ファイル
- `action.yml`: 2つのダウンロードステップを条件付き実行に変更

### テスト確認項目
- ✓ YAML 構文が正しいことを確認
- ☐ ツールが未インストール環境で動作確認（GitHub Actions実行時）
- ☐ ツールが事前インストール済み環境で動作確認（GitHub Actions実行時）
