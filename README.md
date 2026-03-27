# 経理DX 自動化スキル & デモデータ

ウェビナー・デモ動画用のサンプルデータと、Claude Code用スキル（スラッシュコマンド）一式です。

## セットアップ

```bash
# スキルを ~/.claude/skills/ にインストール
for dir in */skill; do
  skill_name=$(dirname "$dir")
  mkdir -p ~/.claude/skills/$skill_name
  cp "$dir/SKILL.md" ~/.claude/skills/$skill_name/
done
echo "✅ 全スキルをインストールしました"
```

### 前提条件

- Claude Code がインストール済み
- Google Sheets API の OAuth トークンが `~/.config/gog/sheets_token.json` に設定済み
- Python 3 + `google-auth`, `google-api-python-client` がインストール済み

---

## スキル一覧

### 1. freee仕訳 × LINE指示 (`/freee-line-journal`)

freeeの未確定取引をLINEから承認・修正指示して仕訳処理を完了するスキル。
スキャンした領収書・請求書をAWS経由でfreeeに連携し、LINE上で仕訳の確認・承認ワークフローを実行する。

**処理フロー:** 書類スキャン → AWS(OCR) → freee未確定取引登録 → LINE通知 → 承認/修正/保留 → freee仕訳確定

| ファイル                                                | 内容                                    |
| ------------------------------------------------------- | --------------------------------------- |
| `freee-line-journal/freee_unconfirmed_transactions.csv` | freee未確定取引12件（仕入/経費/家賃等） |
| `freee-line-journal/line_approval_log.csv`              | LINE承認ログ（承認/修正/保留）          |
| `freee-line-journal/skill/SKILL.md`                     | スキル定義ファイル                      |

**デモコマンド:** `/freee-line-journal ~/Downloads/demo-data/freee-line-journal/`

**効果:** 週8時間 → 週1時間（87.5%削減）、人件費50%削減

---

### 2. 確定申告補助ツール — EC CSV → 勘定科目自動振分 (`/expense-ai`, `/amazon-receipt`)

AmazonなどのEC購入履歴CSVを読み込み、AIが商品名から勘定科目（消耗品費・通信費・書籍費等）を自動推測。確定申告や経費精算用のデータを一括整理してGoogle Spreadsheetに出力する。

**処理フロー:** Amazon CSV読込 → 商品名解析 → 勘定科目推測 → 仕分けルール学習 → Spreadsheet出力

| ファイル                                | 内容                                  |
| --------------------------------------- | ------------------------------------- |
| `amazon-receipt/amazon_orders_2025.csv` | Amazon購入履歴23件（2025年1月〜12月） |
| `amazon-receipt/skill/SKILL.md`         | スキル定義ファイル                    |

**デモコマンド:** `/amazon-receipt ~/Downloads/demo-data/amazon-receipt/amazon_orders_2025.csv`

**効果:** 年間20時間 → 3時間（85%削減）

---

### 3. 確定申告書作成ツール (`/tax-return`)

年間の収入データ・経費データを集約し、所得税の計算と確定申告書のドラフトを自動生成。青色申告特別控除・社会保険料控除・基礎控除を適用した税額計算まで一気通貫で処理する。

**処理フロー:** 収入CSV + 経費CSV → 所得計算 → 控除適用 → 税額算出 → 申告書ドラフト生成 → Spreadsheet出力

| ファイル                       | 内容                                 |
| ------------------------------ | ------------------------------------ |
| `tax-return/income_2025.csv`   | 年間収入データ24件（月別・取引先別） |
| `tax-return/expenses_2025.csv` | 年間経費データ40件（科目別）         |
| `tax-return/skill/SKILL.md`    | スキル定義ファイル                   |

**デモコマンド:** `/tax-return ~/Downloads/demo-data/tax-return/`

**効果:** 年間30時間 → 5時間（83%削減）

---

### 4. 領収書スキャン → データ自動連携 (`/receipt-scan`)

紙の領収書をスキャン→OCRで読み取り、日付・店舗名・金額・勘定科目を自動抽出。freee等の会計ソフトにインポート可能なCSV/仕訳データを生成する。

**処理フロー:** スキャン画像 → OCR解析 → データ抽出(日付/店舗/金額) → 勘定科目推測 → freee CSV生成

| ファイル                            | 内容                              |
| ----------------------------------- | --------------------------------- |
| `receipt-scan/scanned_receipts.csv` | スキャン済み領収書10件（OCR結果） |
| `receipt-scan/skill/SKILL.md`       | スキル定義ファイル                |

**デモコマンド:** `/receipt-scan ~/Downloads/demo-data/receipt-scan/`

**効果:** 月10時間 → 1.5時間（85%削減）

---

### 5. LINE/CW 請求書自動化 (`/invoice-automation`)

LINEやチャットワークで「〇〇社に△万円の請求書出して」と指示するだけで、請求書PDFを自動生成しfreeeに連携。取引先マスタから宛先情報を自動補完し、品目・金額・振込先を整形して出力する。

**処理フロー:** LINE/CW指示 → 取引先マスタ照合 → 請求書PDF生成 → freee連携 → 送付確認通知

| ファイル                               | 内容                         |
| -------------------------------------- | ---------------------------- |
| `invoice-automation/invoice_data.csv`  | 請求書データ6件              |
| `invoice-automation/line_chat_log.csv` | LINEチャットログ（操作履歴） |
| `invoice-automation/skill/SKILL.md`    | スキル定義ファイル           |

**デモコマンド:** `/invoice-automation ~/Downloads/demo-data/invoice-automation/invoice_data.csv`

**効果:** 月8時間 → 1時間（87%削減）

---

### 6. 未入金チェック (`/unpaid-check`)

売掛金台帳と銀行入金データをAIが自動照合。取引先名の表記揺れ（漢字↔カタカナ、(株)↔カ）等）を名寄せし、振込手数料（440円/660円）の差分も考慮してマッチング。未入金リストと督促メールテンプレートを自動生成する。

**処理フロー:** 売掛金台帳 + 入金データ → 名寄せマッチング → 手数料差分吸収 → 未入金特定 → 督促テンプレート生成 → Spreadsheet出力

| ファイル                               | 内容               |
| -------------------------------------- | ------------------ |
| `unpaid-check/accounts_receivable.csv` | 売掛金台帳8件      |
| `unpaid-check/bank_deposits.csv`       | 入金データ5件      |
| `unpaid-check/skill/SKILL.md`          | スキル定義ファイル |

**デモコマンド:** `/unpaid-check ~/Downloads/demo-data/unpaid-check/`

**効果:** 月6時間 → 1時間（83%削減）、未入金見落としゼロ

---

### 7. 月次財務諸表作成 (`/income-statement`)

試算表データから月次推移の損益計算書(P/L)を自動生成。前月比・前年同月比の比較分析と、異常値（大幅な増減）の自動検知・コメント付きでGoogle Spreadsheetに出力する。

**処理フロー:** 試算表CSV → 勘定科目分類 → P/L構成 → 前月比/前年同月比算出 → 異常値検知 → Spreadsheet出力

| ファイル                                    | 内容                                  |
| ------------------------------------------- | ------------------------------------- |
| `income-statement/trial_balance_202603.csv` | 試算表（当月/前月/前年同月の3期比較） |
| `income-statement/skill/SKILL.md`           | スキル定義ファイル                    |

**デモコマンド:** `/income-statement ~/Downloads/demo-data/income-statement/trial_balance_202603.csv`

**効果:** 月16時間 → 4時間（75%削減）

---

### 8. クレカとB/Sの消込作業 (`/reconciliation`)

通帳明細・帳簿データ・クレカ明細の3つを突き合わせ、一致する取引を自動消込。アンマッチ（差異）だけを特定して報告する。日付のずれ（±3日）や金額の端数差も考慮したファジーマッチングを実行。

**処理フロー:** 通帳 + 帳簿 + クレカ明細 → 3者突合 → 日付/金額ファジーマッチ → 消込済み/アンマッチ分類 → Spreadsheet出力

| ファイル                                   | 内容               |
| ------------------------------------------ | ------------------ |
| `reconciliation/bank_statement.csv`        | 通帳データ14件     |
| `reconciliation/ledger_data.csv`           | 帳簿データ20件     |
| `reconciliation/credit_card_statement.csv` | クレカ明細12件     |
| `reconciliation/skill/SKILL.md`            | スキル定義ファイル |

**デモコマンド:** `/reconciliation ~/Downloads/demo-data/reconciliation/`

**効果:** 月10時間 → 1.5時間（85%削減）

---

### 9. freee MCP連携 (`/freee-mcp`)

Claude CodeからMCP(Model Context Protocol)経由でfreee会計APIに直接アクセス。チャットで「今月の売上教えて」「この仕訳登録して」と指示するだけで、freeeの取引データ取得・仕訳登録・勘定科目管理を実行する。

**処理フロー:** Claude Code → MCP → freee API → 取引データ取得/仕訳登録/科目管理

| ファイル                             | 内容                       |
| ------------------------------------ | -------------------------- |
| `freee-mcp/freee_deals_sample.json`  | freee取引データサンプル5件 |
| `freee-mcp/freee_account_items.json` | freee勘定科目マスタ18件    |
| `freee-mcp/skill/SKILL.md`           | スキル定義ファイル         |

**デモコマンド:** `/freee-mcp ~/Downloads/demo-data/freee-mcp/`

---

### 10. 仕訳自動起票 (`/journal-entry`)

発生主義に基づく仕訳を自動起票。固定資産の減価償却・前払費用の按分・給与仕訳等を、借方・貸方の整合性を保ったまま自動作成。複合仕訳にも対応。

**処理フロー:** 取引データ → 勘定科目判定 → 借方/貸方振分 → 整合性チェック → 仕訳帳出力

| ファイル                       | 内容               |
| ------------------------------ | ------------------ |
| `journal-entry/skill/SKILL.md` | スキル定義ファイル |

**デモコマンド:** `/journal-entry`（対話的に取引内容を入力）

---

### 11. AI経費精算 (`/expense-ai`)

領収書画像やCSV購入履歴からAIが勘定科目を自動推測し、Google Spreadsheetに経費精算レポートを出力。過去の仕訳パターンを学習して精度が向上する。

**処理フロー:** 領収書/CSV → AI科目推測 → 過去パターン照合 → 精算レポート生成 → Spreadsheet出力

| ファイル                    | 内容               |
| --------------------------- | ------------------ |
| `expense-ai/skill/SKILL.md` | スキル定義ファイル |

**デモコマンド:** `/expense-ai`（対話的にデータを指定）

---

### 12. 経費異常検知アラート (`/expense-alert`)

経費精算データの異常パターンを自動検知。通常と比べて高額な経費・不自然な頻度・重複申請・休日利用等をAIがスコアリングし、アラートレポートを生成する。

**処理フロー:** 経費データ → 統計分析 → 異常スコアリング → アラート分類(高/中/低) → レポート出力

| ファイル                       | 内容               |
| ------------------------------ | ------------------ |
| `expense-alert/skill/SKILL.md` | スキル定義ファイル |

**デモコマンド:** `/expense-alert`（対話的にデータを指定）

---

### 13. 予実差異分析 (`/variance-analysis`)

予算と実績を自動比較し、差異の要因（ドライバー）をAIがドリルダウン分析。勘定科目別・部門別・月別の差異を可視化し、クライアント報告用レポート原案まで自動生成する。

**処理フロー:** 予算データ + 実績データ → 差異算出 → 要因ドリルダウン → レポート原案生成 → Spreadsheet出力

| ファイル                           | 内容               |
| ---------------------------------- | ------------------ |
| `variance-analysis/skill/SKILL.md` | スキル定義ファイル |

**デモコマンド:** `/variance-analysis`（対話的にデータを指定）

---

## ディレクトリ構成

```
demo-data/
├── README.md                              ← このファイル
├── freee-line-journal/
│   ├── freee_unconfirmed_transactions.csv  ← freee未確定取引データ
│   ├── line_approval_log.csv              ← LINE承認ログ
│   └── skill/SKILL.md                     ← スキル定義
├── amazon-receipt/
│   ├── amazon_orders_2025.csv             ← Amazon購入履歴
│   └── skill/SKILL.md
├── tax-return/
│   ├── income_2025.csv                    ← 年間収入データ
│   ├── expenses_2025.csv                  ← 年間経費データ
│   └── skill/SKILL.md
├── receipt-scan/
│   ├── scanned_receipts.csv               ← スキャン済み領収書
│   └── skill/SKILL.md
├── invoice-automation/
│   ├── invoice_data.csv                   ← 請求書データ
│   ├── line_chat_log.csv                  ← LINEチャットログ
│   └── skill/SKILL.md
├── unpaid-check/
│   ├── accounts_receivable.csv            ← 売掛金台帳
│   ├── bank_deposits.csv                  ← 入金データ
│   └── skill/SKILL.md
├── income-statement/
│   ├── trial_balance_202603.csv           ← 試算表（3期比較）
│   └── skill/SKILL.md
├── reconciliation/
│   ├── bank_statement.csv                 ← 通帳データ
│   ├── ledger_data.csv                    ← 帳簿データ
│   ├── credit_card_statement.csv          ← クレカ明細
│   └── skill/SKILL.md
├── freee-mcp/
│   ├── freee_deals_sample.json            ← freee取引サンプル
│   ├── freee_account_items.json           ← 勘定科目マスタ
│   └── skill/SKILL.md
├── journal-entry/
│   └── skill/SKILL.md
├── expense-ai/
│   └── skill/SKILL.md
├── expense-alert/
│   └── skill/SKILL.md
└── variance-analysis/
    └── skill/SKILL.md
```

## デモ実行の流れ

1. Claude Codeを起動
2. 上記セットアップでスキルをインストール
3. 各スキルのデモコマンドをコピー&ペースト
4. Google Spreadsheetに自動出力される結果を画面共有
5. As Is → To Be の効果を説明

## 効果サマリー

| No  | ツール                 | スキル                | 削減率 | Before       | After        |
| --- | ---------------------- | --------------------- | ------ | ------------ | ------------ |
| 1   | freee × LINE仕訳       | `/freee-line-journal` | 87.5%  | 週8h         | 週1h         |
| 2   | 確定申告補助（EC CSV） | `/amazon-receipt`     | 85%    | 年20h        | 年3h         |
| 3   | 確定申告書作成         | `/tax-return`         | 83%    | 年30h        | 年5h         |
| 4   | 領収書スキャン連携     | `/receipt-scan`       | 85%    | 月10h        | 月1.5h       |
| 5   | LINE/CW請求書自動化    | `/invoice-automation` | 87%    | 月8h         | 月1h         |
| 6   | 未入金チェック         | `/unpaid-check`       | 83%    | 月6h         | 月1h         |
| 7   | 月次財務諸表           | `/income-statement`   | 75%    | 月16h        | 月4h         |
| 8   | クレカ×B/S消込         | `/reconciliation`     | 85%    | 月10h        | 月1.5h       |
| 9   | freee MCP連携          | `/freee-mcp`          | -      | 随時ログイン | チャット即時 |
| 10  | 仕訳自動起票           | `/journal-entry`      | 80%    | 月12h        | 月2.5h       |
| 11  | AI経費精算             | `/expense-ai`         | 85%    | 月8h         | 月1.2h       |
| 12  | 経費異常検知           | `/expense-alert`      | 90%    | 月10h        | 月1h         |
| 13  | 予実差異分析           | `/variance-analysis`  | 75%    | 月20h        | 月5h         |
