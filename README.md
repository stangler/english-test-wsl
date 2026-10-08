# 日→英 テスト — EIGO NO PARTNER

日本語の文を見て英語で答える、日→英専用の練習クイズです。
`translation-test-app` からクラウドインフラ（Vercel / Neon / next-auth / Resend / AI採点）を
すべて除去し、`english-word-typing-app` と同様の **単一HTMLファイル構成** に作り直したものです。

- サーバー不要・DB不要・認証不要・インターネット接続不要（フォント読み込みを除く）
- `english_test.html` をブラウザで開くだけで動作
- 英→日モード・AI採点（Gemini/OpenRouter）・ログインは非搭載

---

## 動作環境（WSL 前提）

本プロジェクトは **WSL（Windows Subsystem for Linux）上に配置して運用** します。

- 以前は Windows のローカルフォルダ（`english-test-win`）で作業していましたが、
  パス・文字コード・Python/uv まわりのエラーが多発したため、WSL 上へ移設しました
- 現在のプロジェクト名および配置先は `english-test-wsl`
- Windows 側フォルダ（`/mnt/c/...`）ではなく、WSL 側のホーム配下
  （例: `~/projects/english-test-wsl`）に置くことで、`build.py` の実行や
  ブラウザからのファイル参照が安定します
- リポジトリは GitHub からクローンして取得します

```bash
git clone https://github.com/stangler/english-test-wsl.git
cd english-test-wsl
```

---

## 使い方

1. `english_test.html` をダブルクリックしてブラウザで開く
2. レッスン・パートを選んで「開始」（必要に応じて「シャッフルする」を有効化）
3. 日本語の問題文を見て英語で入力 → Enter または「判定」
4. テスト後は「間違いのみ再挑戦 / 全部再挑戦」で復習、「履歴を見る」で過去の結果を確認
5. 「📉 苦手分析」で、どの問題をどれだけ間違えているか・どんな傾向で間違えているかを確認

正誤判定はローカルのJSのみで行われます（縮約形展開・冠詞除去・NFKC正規化した上で完全一致比較）。

---

## 収録レッスン

EIGO NO PARTNER の **Starter + Lesson 1〜5** を収録しています（計 725 問）。
Lesson 5 は後から追加したレッスンです。

| レッスン | 問題数 | 主なパート |
| --- | --- | --- |
| Starter | 6 | （パートなし） |
| Lesson 1 | 129 | Part 1〜3, 2-1〜2-3, 学んだことを整理する1/2, Goal Activity, Words & Sounds 1, SPECIAL TOPICS |
| Lesson 2 | 114 | Part 1, 2, 学んだことを整理する1/2, Goal Activity, Words & Sounds 2, Remember? 1 |
| Lesson 3 | 149 | Part 1〜3, 学んだことを整理する1/2, 学んだことを活用する1/2, Goal Activity, Take Action!, SPECIAL TOPICS |
| Lesson 4 | 126 | Part 1, 2, 学んだことを整理する1/2, 学んだことを活用する1/2, Goal Activity, Take Action!, SPECIAL TOPICS |
| **Lesson 5**（追加） | **201** | Part 1〜3, 学んだことを整理する1/2, 学んだことを活用する1/2, Goal Activity, Language Focus, SPECIAL TOPICS, Remember? |

レッスンとパートの一覧はデータ（`json/words-data.js`）から動的に生成されるため、
CSV を更新して再生成すればそのまま選択肢に反映されます。

---

## 履歴と進捗の確認

テスト履歴と出題済みの進捗はブラウザの `localStorage` に保存されます。

- 「📊 履歴を見る」で全履歴を一覧表示
- 「パート別統計」をクリックすると、選択したレッスン/パートの履歴のみに絞り込み
- 「すべて表示」または「✕ クリア」でフィルタを解除
- 上部の「表示回数 / 平均正答率 / 最高記録」はフィルタ後の値を反映
- 開始画面の進捗パネルで、レッスン/パートごとの着手状況（着手済み / 未着手）を確認

保存に使う `localStorage` のキーは次のとおりです。

| キー | 内容 |
| --- | --- |
| `ja2en_quiz_history` | テスト終了時の結果（レッスン / パート / 正解数 / 総数 / 正答率） |
| `ja2en_quiz_attempted` | 出題済みの進捗 |
| `ja2en_quiz_wordstats` | 問題単位の記録（出題回数・間違い回数・誤答・誤答タイプ） |

「🗑 履歴をすべて削除」を実行すると、上記3つがすべて削除されます。

---

## 苦手分析

「📉 苦手分析」（開始画面・結果画面・履歴画面から開けます）で、問題ごとの間違いと傾向を確認できます。
問題単位の記録は **1問答えるたびに保存** されるため、テストを途中でやめた分も残ります。

- **間違い回数ランキング（上位30問）**: 間違い回数 / 出題回数、直近の誤答、誤答タイプ
- **レッスン/パート別の誤答率**
- **誤答タイプ別の件数**
  - 空欄（未回答） / 句読点・記号のミス / 語順違い / 語形違い（-s・-ed・-ing など） / 綴りミス / 語の欠落 / 余分な語 / 別の単語
- **英文の特徴別の誤答率**
  - 疑問文 / 疑問詞で始まる文 / 否定文 / 助動詞を含む / 過去形を含む / 進行形 / 縮約形を含む / 文の長さ（1〜3語・4〜6語・7語以上）
- 全体の誤答率より10ポイント以上高い項目には ▲ が付き、回答3回未満の項目は「データ不足」と表示されます
- 「⬇ JSONエクスポート」で、問題別の記録と履歴をまとめてダウンロードできます

注意点:

- 問題単位の記録は、この機能を追加した後に解いた分から貯まります（それ以前のテスト分は復元できません）
- 誤答タイプと英文の特徴は自動判定のため、あくまで目安です
- 正誤判定は末尾のピリオドなどの記号も含めて完全一致で行うため、記号の抜けは「句読点・記号のミス」に分類されます

---

## ファイル構成

```
english-test-wsl/
├── english_test.html     # アプリ本体（UI + ロジック、単一ファイル）
├── json/
│   └── words-data.js     # window.WORDS = [...] 形式の単語データ（build.pyで生成）
├── csv/
│   └── EIGO_NO_PARTNERに出てくる文.csv  # 出典データ
├── build.py               # csv → json/words-data.js 変換スクリプト
├── pyproject.toml         # プロジェクト設定（uv管理用）
├── uv.lock                # uv のロックファイル
└── README.md
```

## 単語データの再生成

csv を差し替えた場合は再生成してください。

```bash
cd ~/projects/english-test-wsl   # WSL 側のプロジェクトへ移動
uv run python build.py
```

`csv/` 内のCSVファイル（Lesson / Part / 英語 / 日本語の列構成）を読み込み、
`json/words-data.js` を上書き生成します。別解展開（人称代名詞・可能形動詞など）も
`translation-test-app` の `build.py` と同じロジックを流用しています。

---

## translation-test-app との違い

| 項目 | translation-test-app | 本アプリ |
| --- | --- | --- |
| クイズモード | 英→日・日→英 | 日→英のみ |
| フレームワーク | Next.js 14 | なし（静的HTML） |
| DB | PostgreSQL + Prisma | なし（localStorage） |
| 認証 | next-auth + Resend | なし |
| 採点 | 日→英はローカル比較、英→日はAI(Gemini/OpenRouter) | ローカル比較のみ |
| デプロイ | Vercel前提 | ローカルで `english_test.html` を開くだけ（WSL 上に配置） |
| 履歴保存 | あり(DB) | あり（ブラウザの localStorage） |
| 苦手分析 | なし | あり（問題別の誤答・誤答タイプ・英文の特徴別） |

---

## 更新履歴

- **苦手分析を追加**。1問ごとの正誤・誤答を `localStorage`（`ja2en_quiz_wordstats`）に記録し、
  間違い回数ランキング・レッスン/パート別・誤答タイプ別・英文の特徴別の誤答率を確認できるように変更
  （JSONエクスポート付き）
- **Lesson 5 を追加**（201問）。収録範囲が Starter + Lesson 1〜5（計 725 問）に拡大
- **プロジェクト名を `english-test-win` → `english-test-wsl` に変更**
- Windows ローカルで作業するとエラーが多発するため、**WSL 上にプロジェクトを移設**し、
  GitHub（`stangler/english-test-wsl`）からクローンして作業する運用に変更
