---
name: freee-mcp
description: freee会計APIとMCP(Model Context Protocol)を連携。Claude Codeから直接freeeの取引データ取得・仕訳登録・勘定科目管理を実行。「freee MCP」「freee連携」「freee API」等でトリガー。
user_invocable: true
---

# freee MCP連携スキル

Claude CodeからfreeeのAPIにMCP経由で直接アクセスし、取引データの取得・仕訳の登録・勘定科目の管理を実行します。

---

## クイックスタート

```
/freee-mcp <操作指示>
```

**入力例:**

- `/freee-mcp 今月の取引一覧を取得して` - 取引データ取得
- `/freee-mcp 3月分の仕訳を一括登録 ~/Downloads/journal.csv` - 仕訳CSV登録
- `/freee-mcp 売掛金の残高を確認` - 勘定科目残高照会
- `/freee-mcp` - 対話的に操作を選択

---

## 対応操作

### 取引データ

| 操作         | freee API                 | 説明                     |
| ------------ | ------------------------- | ------------------------ |
| 取引一覧取得 | GET /api/1/deals          | 期間指定で取引を一覧取得 |
| 取引登録     | POST /api/1/deals         | 新規取引を登録           |
| 取引更新     | PUT /api/1/deals/{id}     | 既存取引を更新           |
| 取引検索     | GET /api/1/deals (filter) | 取引先・金額・日付で検索 |

### 仕訳

| 操作         | freee API                       | 説明                       |
| ------------ | ------------------------------- | -------------------------- |
| 仕訳帳取得   | GET /api/1/journals             | 仕訳帳データをダウンロード |
| 振替伝票登録 | POST /api/1/manual_journals     | 振替伝票（手動仕訳）を登録 |
| 振替伝票更新 | PUT /api/1/manual_journals/{id} | 既存仕訳を修正             |

### 勘定科目・マスタ

| 操作       | freee API                | 説明                 |
| ---------- | ------------------------ | -------------------- |
| 科目一覧   | GET /api/1/account_items | 勘定科目マスタを取得 |
| 取引先一覧 | GET /api/1/partners      | 取引先マスタを取得   |
| 税区分一覧 | GET /api/1/taxes/codes   | 税区分マスタを取得   |

### レポート

| 操作     | freee API                             | 説明               |
| -------- | ------------------------------------- | ------------------ |
| 試算表   | GET /api/1/reports/trial_bs_two_years | 貸借対照表を取得   |
| P/L      | GET /api/1/reports/trial_pl_two_years | 損益計算書を取得   |
| 残高確認 | GET /api/1/reports/trial_bs           | 勘定科目残高を確認 |

---

## MCP連携アーキテクチャ

```
[Claude Code] ←MCP→ [freee MCPサーバー] ←REST API→ [freee会計]
                           ↓
                     [OAuth2認証]
                           ↓
                   [freee APIトークン]
```

### MCPサーバー設定

```json
{
  "mcpServers": {
    "freee": {
      "command": "node",
      "args": ["path/to/freee-mcp-server/index.js"],
      "env": {
        "FREEE_ACCESS_TOKEN": "your_token",
        "FREEE_COMPANY_ID": "your_company_id"
      }
    }
  }
}
```

---

## 処理フロー

### Step 1: 操作の識別

ユーザーの指示からfreee APIの操作を特定:

| 指示例                 | API操作                         |
| ---------------------- | ------------------------------- |
| 「今月の取引を見せて」 | GET /api/1/deals?start_date=... |
| 「この仕訳を登録して」 | POST /api/1/manual_journals     |
| 「売掛金の残高は？」   | GET /api/1/reports/trial_bs     |
| 「取引先ABCの取引」    | GET /api/1/deals?partner_id=... |

### Step 2: API実行 & データ取得

- freee APIを呼び出し、JSONレスポンスを取得
- データを構造化してGoogle Spreadsheetに出力

### Step 3: Google Spreadsheet出力

操作に応じたシート構成で出力:

**取引一覧の場合:**

1. **取引一覧**: 日付, 取引先, 勘定科目, 金額, 摘要
2. **月次集計**: 月別の取引件数・金額合計

**残高照会の場合:**

1. **勘定科目残高**: 科目コード, 科目名, 借方残高, 貸方残高
2. **残高推移**: 月次の残高推移

---

## 必要な認証情報

- freee API OAuth2トークン
- freee事業所ID
- Google Sheets API: `~/.config/gog/sheets_token.json`

---

## As Is → To Be

| 項目       | As Is                        | To Be                        |
| ---------- | ---------------------------- | ---------------------------- |
| データ取得 | freeeにログイン→画面操作     | Claude Codeで自然言語指示    |
| 仕訳登録   | 1件ずつ画面入力              | CSV一括登録/自然言語指示     |
| レポート   | 画面でエクスポート→Excel加工 | 自動取得→分析付きSpreadsheet |
| 工数       | 随時ログイン必要             | チャット感覚で即時実行       |
