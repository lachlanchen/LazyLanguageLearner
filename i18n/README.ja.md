[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# lazylanguagelearner

言語を怠け者流で学ぼう。

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

## 🌍 概要

LazyLanguageLearner は、以下を組み合わせた Python ベースの言語学習ワークフローです。

- Rosetta Stone コース教材ドキュメントの PDF 取得
- PDF 解析と文抽出を行い CSV データセット化
- OpenAI を使った多言語文変換（発音ペア付き）とローカルディスクキャッシュ
- ルビ用の発音注釈付き多言語テキストを表示する軽量 Tornado Web アプリ

現在のリポジトリはスクリプト駆動型です（まだ pip モジュールとしてはパッケージ化されていません）。データファイルとノートブックはリポジトリ内に直接含まれています。

## ✨ 特徴

- `rs_html.py` に埋め込まれたリンクから言語コース PDF をダウンロード（`download_course_text.py`）
- PDF からセクション/文データを抽出し、構造化された CSV を生成（`pdf_to_csv.py`, `language_extraction.py`）
- OpenAI のプロンプト/レスポンスを `cache/*.json` にキャッシュし、API の重複利用を削減（`openai_request.py`）
- AI レスポンスを JSON に解析（リトライロジックとカスタム JSON 解析エラーを含む）
- `translations.json` から多言語文ブロックを Tornado 経由で配信（`app.py` + `templates/index.html`）
- 描画前に日本語の発音正規化（`katakana` から `hiragana`）を実施

## 🗂️ プロジェクト構成

```text
.
├── README.md
├── LICENSE
├── app.py
├── download_course_text.py
├── rs_html.py
├── pdf_extraction.py
├── pdf_to_csv.py
├── language_extraction.py
├── openai_request.py
├── multilingual_sentence.py
├── translations.json
├── japanese_language_data.csv
├── japanese_language_data copy.csv
├── processed_words.csv
├── templates/
│   └── index.html
├── cache/
│   └── *.json
├── i18n/
│   └── (currently empty)
└── *.ipynb
```

## ✅ 前提条件

前提（現時点で lockfile や依存マニフェストがコミットされていないため）：

- Python 3.10+（近いバージョンでも動作する可能性は高いですが、正確な検証マトリクスは未記載です）
- `pip` と `venv`
- モデル利用スクリプト向けの OpenAI API キー

import から推測される Python 依存関係：

| Package | Used by |
|---|---|
| `tornado` | `app.py` の Web サーバー |
| `openai` | `openai_request.py` の API 呼び出し |
| `PyPDF2` | 抽出スクリプトの PDF 解析 |
| `requests` | `download_course_text.py` の PDF ダウンロード |
| `beautifulsoup4` | ダウンローダーの HTML 解析 |

## 🛠️ インストール

```bash
# from repository root
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

## 🚀 使い方

### 1) 元となる PDF をダウンロード

```bash
python download_course_text.py
```

これにより `downloaded_pdfs/` が作成され、言語/ユニットの PDF が保存されます。

### 2) 日本語 PDF コンテンツを CSV に抽出

```bash
python pdf_to_csv.py
```

現在のスクリプトの既定出力: `japanese_language_data.csv`。

### 3) （任意）セクション/ページ/文テキストを対話的に切り出し

```bash
python language_extraction.py
```

このスクリプトには編集可能なサンプル変数（`level`, `section`, `sentence_num`）があり、抽出テキストを表示します。

### 4) OpenAI フローで多言語 JSON を生成

```bash
python multilingual_sentence.py
```

現在の挙動メモ：

- 最初の CSV 行のみ処理されます（ループ内に `break` があるため）。
- スクリプトは現在、プロンプト生成時に未定義変数（`japanese_text`）を参照しているため、安定運用には小さな修正が必要です。

### 5) Web アプリを起動

```bash
python app.py
```

- Tornado はポート `7788` で待ち受けます。
- ブラウザで開く: `http://localhost:7788/`。
- 注: 起動ログには現在 `http://localhost:8888` と表示されますが、実際のバインド先は `7788` です。

## ⚙️ 設定

環境変数：

| Variable | Required | Purpose | Current default |
|---|---|---|---|
| `OPENAI_API_KEY` | Yes | `OpenAI()` クライアント初期化に必須 | N/A |
| `OPENAI_MODEL` | No | チャットモデルの任意上書き | `gpt-4-0125-preview` |

実行時ファイル/ディレクトリ：

- `downloaded_pdfs/`: ダウンローダーが作成し、抽出スクリプトが利用
- `cache/`: OpenAI 呼び出しのリクエスト/レスポンスキャッシュ
- `translations.json`: Tornado UI 描画のデータソース

## 🧾 データ形式の例

### CSV (`japanese_language_data.csv`)

`pdf_to_csv.py` が使うヘッダー：

```csv
Level,Unit,Section,Sentence No.,Content
```

### JSON (`translations.json`)

Web UI は、`part` と `phonetic` を含む `pairs` エントリを持つ言語キーを想定しています。

```json
{
  "ja": {
    "full": "...",
    "pairs": [
      { "part": "日", "phonetic": "ひ" }
    ]
  },
  "en": { "full": "...", "pairs": [] },
  "ar": { "full": "...", "pairs": [] },
  "zh": { "full": "...", "pairs": [] },
  "yue": { "full": "...", "pairs": [] }
}
```

## 🧪 開発メモ

- 現在このリポジトリには `requirements.txt`、`pyproject.toml`、CI ワークフローがありません。
- スクリプトはリポジトリルートから直接実行する設計です。
- 既存ノートブック（`*.ipynb`）は探索/試作寄りに見えます。
- 大きな CSV 成果物は Git に直接バージョン管理されています。
- `i18n/` は存在しており、翻訳 README バリアントの配置準備ができています。

## 🩺 トラブルシューティング

- `ModuleNotFoundError`: 有効化した仮想環境に推測依存関係をインストールしてください。
- `OPENAI` 認証エラー: `OPENAI_API_KEY` がシェルで export 済みか確認してください。
- `FileNotFoundError: downloaded_pdfs`: 先に `python download_course_text.py` を実行してください。
- `multilingual_sentence.py` の `japanese_text` 失敗: プロンプト生成時の `japanese_text` を `content` に置き換えてください。
- アプリのポート混乱: `app.listen(...)` を変更していない限り `http://localhost:7788/` を使用してください。

## 🛣️ ロードマップ

プロジェクトの次の候補ステップ：

- 依存マニフェスト（`requirements.txt` または `pyproject.toml`）を追加
- `multilingual_sentence.py` のプロンプト変数を修正し、1行のみ処理する `break` を除去してバッチ処理化
- Tornado 起動時の表示 URL と実際のバインドポートを一致させる
- PDF 抽出の正規表現挙動、および JSON 解析/リトライロジックのテストを追加
- ファイル内編集を減らすため、言語/レベル/パス用の CLI 引数を追加
- `i18n/` に README 翻訳ファイルを追加

## 🤝 コントリビュート

コントリビューション歓迎です。

1. リポジトリを Fork します。
2. 機能ブランチを作成します。
3. 焦点を絞った変更を、明確なコミットメッセージで行います。
4. 何をなぜ変更したかを説明した Pull Request を作成します。

抽出ロジックを変更する場合は、レビューしやすいよう入出力サンプルを添えてください。

## 🙏 謝辞

- Rosetta Stone のコース教材リンクは `rs_html.py` に埋め込まれており、PDF ダウンロードの参照元として利用しています。
- OpenAI API は多言語生成と発音構造化に使用しています。

## 📄 ライセンス

本プロジェクトは Apache License 2.0 の下でライセンスされています。詳しくは [LICENSE](LICENSE) を参照してください。
