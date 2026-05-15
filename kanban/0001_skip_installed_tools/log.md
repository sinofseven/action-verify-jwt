# タスク 0001: インストール済みツールのダウンロードステップをスキップ - 実装ログ

## ヘッダー情報
- **タスクID**: 0001
- **開始時刻**: 2026-05-15T15:30:00+09:00
- **ステータス**: 実装中

## タスク概要
`Download jwt-cli`ステップと`Download oidc-jwks-converter`ステップを既にインストール済みだったらスキップするようにしてください。

## 調査結果

### ファイル構造
- リポジトリは GitHub Action の実装
- 実装スクリプトは外部ファイルに存在せず、**すべてのステップが `action.yml` 内にインラインで実装**

### action.yml の構成
- **実行方式**: `using: composite` - 複合アクション形式
- **ステップ数**: 5 つ

### 対象ステップの詳細

#### Download jwt-cli（行30-42）
```yaml
- name: Download jwt-cli
  working-directory: ${{ runner.temp }}/${{ env.EXECUTION_UUID }}/jwt-cli
  shell: bash
  env:
    DESTINATION_DIR: ${{ runner.temp }}/${{ env.EXECUTION_UUID }}/bin/
    PACKAGE_URL: https://github.com/mike-engel/jwt-cli/releases/download/6.2.0/jwt-linux-musl.tar.gz
  run: |
    FILENAME=$(basename "${PACKAGE_URL}")
    curl -OL "${PACKAGE_URL}"
    tar xf "${FILENAME}"
    mv jwt "${DESTINATION_DIR}"
```

#### Download oidc-jwks-converter（行44-57）
```yaml
- name: Download oidc-jwks-converter
  working-directory: ${{ runner.temp }}/${{ env.EXECUTION_UUID }}/oidc-jwks-converter
  shell: bash
  env:
    DESTINATION_DIR: ${{ runner.temp }}/${{ env.EXECUTION_UUID }}/bin/
    PACKAGE_URL: https://github.com/sinofseven/oidc-jwks-converter/releases/download/v0.1.0/oidc-jwks-converter_v0.1.0_x86_64-unknown-linux-musl.zip
  run: |
    FILENAME=$(basename "${PACKAGE_URL}")
    STEM=${FILENAME%.zip}
    curl -OL "${PACKAGE_URL}"
    unzip "${FILENAME}"
    mv "${STEM}/oidc-jwks-converter" "${DESTINATION_DIR}"
```

### 現状
- 両ステップとも無条件で常に実行
- ツールのインストール状態をチェックする仕組みなし

## 実装プラン

### アプローチ
各ダウンロードステップに**事前チェック**を追加し、ツールがすでに PATH 上に存在する場合はダウンロードをスキップ。

```bash
# jwt-cli の場合
if command -v jwt >/dev/null 2>&1; then
  echo "jwt-cli is already installed, skipping download"
else
  # ダウンロード処理（既存コード）
fi

# oidc-jwks-converter の場合
if command -v oidc-jwks-converter >/dev/null 2>&1; then
  echo "oidc-jwks-converter is already installed, skipping download"
else
  # ダウンロード処理（既存コード）
fi
```

### 修正対象
- `action.yml` の Download jwt-cli ステップ
- `action.yml` の Download oidc-jwks-converter ステップ

## プランニング経緯
初回提案がそのまま承認された。

## 実装ステップと進行状況

### ステップ 1: Download jwt-cli を修正
- ステータス: 実装予定
- 修正内容: `command -v jwt` でチェック、未検出時にダウンロード実行

### ステップ 2: Download oidc-jwks-converter を修正
- ステータス: 実装予定
- 修正内容: `command -v oidc-jwks-converter` でチェック、未検出時にダウンロード実行

### ステップ 3: 動作確認
- ステータス: 実装予定
- 確認内容: 修正後の action.yml が YAML として正しく整形されていることを確認

## 編集したファイル

### /Users/yuta/space/private/action-verify-jwt/action.yml
- 修正ステップ: Download jwt-cli（行30-42）
  - 変更内容: `if command -v jwt` チェックを追加
  - 理由: ツールが既にインストール済みの場合、ダウンロード処理をスキップして実行時間を短縮
  
- 修正ステップ: Download oidc-jwks-converter（行44-57）
  - 変更内容: `if command -v oidc-jwks-converter` チェックを追加
  - 理由: ツールが既にインストール済みの場合、ダウンロード処理をスキップして実行時間を短縮

## 実行したコマンド

1. `action.yml` の `Download jwt-cli` ステップを修正
   - `if command -v jwt >/dev/null 2>&1; then` を追加
   - 既存のダウンロード処理をelse句に移行

2. `action.yml` の `Download oidc-jwks-converter` ステップを修正
   - `if command -v oidc-jwks-converter >/dev/null 2>&1; then` を追加
   - 既存のダウンロード処理をelse句に移行

3. `git diff action.yml` で修正内容を確認

## エラー・問題

なし - 修正は正常に完了

## 完了日時

2026-05-15T15:35:00+09:00
