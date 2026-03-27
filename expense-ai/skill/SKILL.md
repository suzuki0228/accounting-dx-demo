---
name: expense-ai
description: 領収書画像やCSV購入履歴からAIが勘定科目を自動推測し、Google Spreadsheetに経費精算レポートを出力。「経費精算」「領収書」「勘定科目」「expense」等でトリガー。
user_invocable: true
---

# AI経費精算スキル

領収書画像・購入履歴CSV・法人カード明細から、AIが勘定科目を自動推測してGoogle Spreadsheetに経費精算レポートを出力します。

---

## クイックスタート

```
/expense-ai <入力ソースのパスまたは指示>
```

**入力例:**
- `/expense-ai ~/請求書_amazon/amazon_orders_2025.csv` - Amazon購入CSV
- `/expense-ai ~/Downloads/receipts/` - フォルダ内の領収書画像を一括処理
- `/expense-ai ~/Downloads/ai-expense-demo/` - デモデータで実行
- `/expense-ai` - 対話的に入力ソースを確認

---

## 神技5選デモ

ウェビナー・動画用のライブデモとして、以下の5つの「神技」を実演できます。

### 神技1: 領収書画像の一括読み取り
- A4用紙に貼り付けた複数の領収書を1枚の画像で認識
- Claude Codeの画像解析で店名・日付・金額・品目を自動抽出
- **デモ手順:** `receipts/a4_multi.png` をClaude Codeに読ませるだけ

### 神技2: 品目→勘定科目AI推測（単体デモあり）
- 品目名のキーワードマッチングで勘定科目を自動分類
- 10万円以上の備品は資産計上フラグ、交際費は1人5,000円超の警告
- **単体デモスクリプト:**
  ```bash
  cd ~/Downloads/ai-expense-demo && python3 scripts/demo2_gsheet.py
  ```
- **出力:** Google Spreadsheet 2シート（勘定科目AI推測 + 勘定科目別集計）

### 神技3: 二重申請チェック
- Web領収書と紙レシートの重複を自動検知
- 金額一致 AND 日付±1日 AND 支払先名の類似度で判定
- **デモスクリプト:**
  ```bash
  python3 ~/Downloads/ai-expense-demo/scripts/demo3_duplicate_check.py
  ```

### 神技4: 未提出者への自動催促
- 法人カード明細と提出済み領収書を突合→未提出リスト生成
- Slack風のリマインドメッセージをモック表示
- **デモスクリプト:**
  ```bash
  python3 ~/Downloads/ai-expense-demo/scripts/demo4_slack_reminder.py
  ```

### 神技5: インボイスT番号照合
- 適格請求書のT番号（13桁）を国税庁APIで照会
- 事業者名・登録状態の検証
- **デモスクリプト:**
  ```bash
  python3 ~/Downloads/ai-expense-demo/scripts/demo5_t_number_api.py T2021001052319
  ```

### 全神技まとめスプレッドシート
5つの神技を1つのGoogle Spreadsheetにまとめて出力:
```bash
cd ~/Downloads/ai-expense-demo && python3 scripts/write_demo_to_gsheet.py
```

---

## 処理フロー

### Step 1: 入力データの識別

入力ソースに応じて処理を分岐:

| 入力タイプ | 拡張子/形式 | 処理方法 |
|-----------|------------|---------|
| 領収書画像 | .png, .jpg, .pdf | Claude Codeの画像読み取りでOCR→データ抽出 |
| 購入履歴CSV | .csv | pandas/csvで読み込み→品目ごとに分類 |
| デモディレクトリ | ai-expense-demo/ | デモスクリプト実行 |
| 法人カード明細 | .json, .csv | 突合・未提出チェック付き |

### Step 2: 勘定科目の自動推測

品目名のキーワードマッチングで勘定科目を推測:

| キーワード例 | 勘定科目 | コード |
|------------|---------|--------|
| コピー用紙, ボールペン, 付箋, トナー | 事務用品費 | 6301 |
| モニター, キーボード, PC, プリンター | 消耗品費（備品） | 6302 |
| お茶, コーヒー, 水, おにぎり | 会議費 | 6352 |
| 宴会, コース, 飲み放題 | 交際費 | 6361 |
| タクシー, 電車, 航空券, ANA | 旅費交通費 | 6321 |
| ホテル, 宿泊 | 旅費交通費（宿泊） | 6322 |
| 手袋, ヘルメット, 洗剤 | 消耗品費 | 6303 |
| レンチ, ドライバー, 工具 | 消耗品費（工具） | 6304 |
| 書籍, Kindle, 入門, 教科書 | 新聞図書費 | 6371 |
| サブスクリプション, Unlimited | 通信費（サブスク） | 6341 |

**特殊ルール:**
- 10万円以上の備品 → 「工具器具備品（資産）」(1600) として資産計上フラグ
- 交際費で1人5,000円超 → 損金不算入の可能性を警告
- 注文番号がD01-で始まる → Kindle書籍として「新聞図書費」
- 自動分類できない → 「【要確認】」として手動確認フラグ

### Step 3: Google Spreadsheet出力

カスタムマッピングファイルがある場合は `data/account_mapping.json` を参照。

**出力シート構成（CSVの場合）:**
1. **購入明細_勘定科目推測**: 全品目 + AI推測結果
2. **勘定科目別集計**: 科目ごとの件数・金額・構成比
3. **月別推移**: 月×科目のクロス集計

**出力シート構成（デモ全体の場合）:**
1. **神技1_領収書読取**: 画像OCR結果（品目・金額・税率）
2. **神技2_勘定科目推測**: AI分類結果
3. **神技3_二重申請チェック**: 領収書とカード明細の突合
4. **神技4_未提出リスト**: 未提出者の一覧 + 催促文テンプレート
5. **神技5_インボイスT番号**: 適格請求書の登録番号検証

**出力シート構成（神技2単体の場合）:**
1. **勘定科目AI推測**: 21品目のAI分類結果（日付・取引先・品目・金額・税率・勘定科目・推測根拠）
2. **勘定科目別集計**: 7科目の件数・金額・構成比

### Step 4: 書式設定

- ヘッダー行: 色分け（シートごとに異なるテーマカラー）、太字、中央揃え
- ヘッダー固定（スクロール時も表示）
- 金額列: 円マーク+カンマ区切り (¥#,##0)
- 勘定科目列: 太字+青文字
- 条件付き書式: 「要確認」「未提出」行を赤ハイライト
- 列幅自動調整

---

## 必要な認証情報

Google Sheets API の OAuth トークンが必要:
- トークンパス: `~/.config/gog/sheets_token.json`
- クライアント情報: `~/.config/gog/credentials.json`
- トークン失効時は自動でブラウザ認証フローを起動
- スコープ: `https://www.googleapis.com/auth/spreadsheets`

**初回セットアップ:**
1. Google Cloud Console で OAuth 2.0 クライアントIDを作成（デスクトップアプリ）
2. credentials.json を `~/.config/gog/` に配置
3. スクリプト初回実行時にブラウザが開き、Google認証を完了
4. トークンが `sheets_token.json` に自動保存される

---

## ディレクトリ構成

```
~/Downloads/ai-expense-demo/
├── receipts/                          # ダミー領収書画像（6種）
│   ├── convenience.png                # セブン-イレブン（おにぎり・水・ボールペン）
│   ├── amazon_detail.png              # Amazon（コピー用紙・4Kモニター・お茶）
│   ├── restaurant.png                 # 居酒屋（宴会コース・飲み放題）
│   ├── monotaro_detail.png            # MonotaRO（手袋・工具・洗剤）
│   ├── travel_web.png                 # ANA出張パック（航空券+ホテル）
│   ├── invoice_t_number.png           # T番号付きインボイス
│   └── a4_multi.png                   # A4に5枚貼り付けスキャン風
├── data/
│   ├── account_mapping.json           # 品目→勘定科目マッピングルール
│   ├── card_statements.json           # 法人カード利用明細（7件）
│   └── submitted_receipts.json        # 提出済み領収書（3件）
├── scripts/
│   ├── generate_receipts.py           # HTML→PNG変換（Playwright）
│   ├── write_demo_to_gsheet.py        # 全5神技→GSheet（5シート）
│   ├── demo2_gsheet.py                # 神技2単体→GSheet（2シート）
│   ├── demo3_duplicate_check.py       # 神技3: 二重申請チェック
│   ├── demo4_slack_reminder.py        # 神技4: 未提出者Slack催促モック
│   └── demo5_t_number_api.py          # 神技5: 国税庁T番号API照会
├── templates/                         # 領収書HTMLテンプレート
└── requirements.txt
```

---

## 関連スクリプト

| スクリプト | 用途 | 出力 |
|-----------|------|------|
| `scripts/write_demo_to_gsheet.py` | 全5神技→GSheet | 5シートのスプレッドシート |
| `scripts/demo2_gsheet.py` | 神技2単体→GSheet | 2シートのスプレッドシート |
| `scripts/demo3_duplicate_check.py` | 二重申請チェック | ターミナル出力 |
| `scripts/demo4_slack_reminder.py` | 未提出者催促モック | ターミナル出力 |
| `scripts/demo5_t_number_api.py` | T番号API照会 | ターミナル出力 |
| `~/請求書_amazon/write_to_gsheet.py` | Amazon CSV→GSheet | 3シートのスプレッドシート |

---

## 生成済みスプレッドシート（実績）

| スプレッドシート名 | 内容 | ID |
|------------------|------|-----|
| Amazon購入履歴_勘定科目AI推測_2025 | Amazon CSV 123件→3シート | `10N5Pt1ZyWWn2bKwreGZmUm9IulJZdfnw0YG72LGAOLQ` |
| AI経費精算デモ_神技5選_レポート | 全5神技→5シート | `16EujTFgZtIeK-Nk6o8QFTJ0T-U4gj3LGIBmIv8nc5kQ` |
| 神技2_勘定科目AI推測デモ | 神技2単体→2シート | `18g_d2zRtk6VvKbB9sPwP7ynhouPczCOVWlgHz25KgSE` |

---

## 拡張ポイント

- `account_mapping.json` にルールを追加すれば分類精度向上
- 法人カード明細CSV対応（銀行フォーマット）
- freee/マネーフォワード連携（API）
- 国税庁T番号API照合の自動化
- Slack Webhook連携（モック→本番切替）
