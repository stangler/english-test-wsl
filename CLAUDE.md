# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

日本語の文を見て英語で答える **日→英専用クイズアプリ**（EIGO NO PARTNER）。
サーバー不要・単一HTMLファイルで動作。ブラウザで `english_test.html` を開くだけで使用可能。

- プロジェクト名: `english-test-wsl`（旧 `english-test-win`）
- 収録範囲: **Starter + Lesson 1〜5（計 725 問）**。Lesson 5 は後から追加したレッスン
- 実行環境: **WSL 上のホーム配下**（例: `~/projects/english-test-wsl`）。
  Windows ローカルのフォルダで作業するとエラーが多発したため WSL へ移設済み（詳細は下記）

## 動作環境（WSL 前提）

- Windows 側のパス（`/mnt/c/...`）ではなく、WSL 側のホーム配下にリポジトリを置く
- リポジトリは GitHub（`stangler/english-test-wsl`）からクローンして取得

```bash
git clone https://github.com/stangler/english-test-wsl.git
cd english-test-wsl
```

## 重要なファイル

- `english_test.html` — アプリ本体（996行）。UI（CSS）、データ読込、クイズロジック、履歴管理が単一ファイルに収まっている
- `json/words-data.js` — `window.WORDS` 配列（725件）。csv から build.py で生成される単語データ
- `build.py` — CSV → JSデータ変換スクリプト（190行・標準ライブラリのみ）
- `csv/EIGO_NO_PARTNERに出てくる文.csv` — 出典データ（Lesson / Part / 英語 / 日本語 の4列構成）
- `.python-version` / `pyproject.toml` / `uv.lock` — Python 3.13 を uv で管理
- `README.md` — 利用者向けの説明（環境・レッスン一覧・再生成手順）

> Node.js / pnpm / Dev Container（`.devcontainer/`）は **このリポジトリでは使用していない**。
> ビルドツールもフレームワークも無く、Python（uv）と素の HTML のみで完結する。

## コマンド

| 操作 | コマンド |
|------|----------|
| 単語データの再生成 | `uv run python build.py` |
| ロックファイルの更新 | `uv lock` |
| 環境の同期 | `uv sync` |

- Python 環境は `uv` で管理（`.python-version` = 3.13、`pyproject.toml`、`uv.lock`）
- 依存: なし（標準ライブラリ csv のみ）
- `uv run python build.py` は初回実行時に `.venv` を自動作成する（`.venv` は `.gitignore` 済み）
- **`pnpm install` / `npm install` は不要**（package.json は存在しない）

## アーキテクチャ

### データフロー

```
csv/出典データ → build.py → json/words-data.js → english_test.html (window.WORDS)
```

1. `build.py` が `csv/` 内のCSVを読み、別解展開（人称代名詞・可能形動詞など）を行った上で `window.WORDS = [...]` 形式のJSファイルを生成
2. `english_test.html` が `<script src="json/words-data.js">` でデータを読込

### english_test.html の構成（1ファイル・996行）

- **CSS（9-185行）**: `.note-page` ベースのノート風UI。変数定義は `:root`
- **HTML（187-190行）**: `<div class="note-page" id="app">` にマウント
- **データ読込（192行）**: `<script src="json/words-data.js">` → `const WORDS = window.WORDS || []`
- **JS ロジック（193-994行）**:
  - 正規化・比較: `normalizeEn()`, `compareWords()` — 縮約形展開・NFKC正規化・冠詞除去・空白正規化後、完全一致比較
  - 状態管理: `state` オブジェクト（`screen`, `selectedLesson`, `queue`, `currentIndex` など）
  - 描画: `render()` → 画面種別に応じて `renderStart()` / `renderQuiz()` / `renderResult()` / `renderHistory()`
  - 履歴: `localStorage` に保存（`ja2en_quiz_history` キー）
  - 進捗: `localStorage` に `ja2en_quiz_attempted` キーで出題済み問題を記録
  - チャート: SVG で直接描画（`renderChart()`）

### 画面遷移

```
start (レッスン/パート選択 + 進捗パネル) → quiz (問題表示・回答) → result (結果表示) → history (履歴一覧 + パート別統計)
```

- start の進捗パネル: `renderProgressPanel()` — 履歴と `ja2en_quiz_attempted` からレッスン/パートごとの
  着手状況・正答率をバッジ表示（`getProgressGroups()` / `getBadgeStatus()`）
- history のパート別統計: バッジクリックでフィルタを適用（`state.currentHistoryFilter`）し、
  該当レッスン/パートの履歴のみ表示。「すべて表示」「✕ クリア」で解除
- チャート: `renderChart()` / `renderPerGroupCharts()` が SVG を直接描画

### 別解展開（build.py 側）

- 人称代名詞: `私は` ↔ `ぼくは` / `僕は` / `ぼくが` / `僕が`
- 可能形: `...ことができます` → `...できる` などの縮約形
- 特殊パターン: `...をみます` → `...を見ます` など

## 収録レッスン（CSV の `Lesson` 列）

| レッスン | 問題数 | 主なパート |
| --- | --- | --- |
| Starter | 6 | （パートなし・`part` は空文字） |
| 1 | 129 | 1〜3, 2-1〜2-3, 学んだことを整理する1/2, Goal Activity, Words & Sounds 1, SPECIAL TOPICS |
| 2 | 114 | 1, 2, 学んだことを整理する1/2, Goal Activity, Words & Sounds 2, Remember? 1 |
| 3 | 149 | 1〜3, 学んだことを整理する1/2, 学んだことを活用する1/2, Goal Activity, Take Action!, SPECIAL TOPICS |
| 4 | 126 | 1, 2, 学んだことを整理する1/2, 学んだことを活用する1/2, Goal Activity, Take Action!, SPECIAL TOPICS |
| 5 | 201 | 1〜3, 学んだことを整理する1/2, 学んだことを活用する1/2, Goal Activity, Language Focus, SPECIAL TOPICS, Remember? |

- レッスン/パートの選択肢は `english_test.html` 側にハードコードされていない。
  `const lessons = [...new Set(WORDS.map(w => w.lesson))]` と `getLessonParts()` で動的生成される
- 表示ラベルは `Starter` のみ特別扱い、それ以外は `Lesson N` として表示

## 編集時の注意点

- `english_test.html` は **単一ファイル・フレームワークなし**。React やビルドツールは使われていない
- CSS は `.note-page` 配下にカプセル化。変数(`--bg`, `--red` など)でテーマ色を管理
- `window.WORDS` の構造 (`lesson`, `part`, `en`, `ja`, `ja_answers`) は build.py 側と固定
- 正誤判定は **ローカルJSのみ**（AI採点なし）
- 履歴はブラウザの `localStorage` に保存され、サーバー送信しない
- 問題を追加・修正する場合は **CSV を編集 → `uv run python build.py` で再生成**。
  `json/words-data.js` は生成物なので直接編集しない（レッスン/パートの選択肢も自動反映される）
- CSV は UTF-8（BOM なし）で、build.py は `encoding="utf-8-sig"` で読み込む。
  Excel で保存し直すと文字化けや列ずれが起きるため、差し替え後は再生成して `git diff` を確認する
- 作業は **WSL 側のホーム配下**で行う。`/mnt/c/...`（Windows 側）に置くと uv / Python 実行時に
  エラーが出やすいため、リポジトリは WSL 側に配置する