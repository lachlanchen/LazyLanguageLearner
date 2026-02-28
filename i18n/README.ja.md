[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| 属性 | 値 |
|---|---|
| 種別 | スクリプト駆動の多言語言語学習パイプライン |
| 実行環境 | Python CLI + Tornado Webアプリ |
| 主なソース | Rosetta Stone コースPDF |
| ストレージ | ローカルのCSV + JSONキャッシュファイル |
| デフォルトポート | `7788` |

LazyLanguageLearner は、言語コースのPDFを再利用可能な多言語学習データに変換し、最小限のWeb UIで表示するスクリプト駆動のPythonワークフローです。

## 🌍 概要

このリポジトリは、コンテンツ抽出・変換・提供を組み合わせています。

| 手順 | 目的 |
|---|---|
| 1 | `rs_html.py` に埋め込まれたリンクからRosetta StoneコースPDFをダウンロードします。 |
| 2 | PDFを文単位のCSV行にパースします。 |
| 3 | OpenAIを使って多言語/発音付きのバリアントを生成し、ディスク上でキャッシュします。 |
| 4 | 整形された文をTornado Web UIで発音注釈付きでレンダリングします。 |

このプロジェクトは意図的に軽量でルート中心です。スクリプトはインストール済みパッケージとしてではなく、リポジトリルートから直接実行する前提で設計されています。

## ✨ 特徴

- `download_course_text.py` を使って、`rs_html.py` に埋め込まれたリンクからPDFを自動ダウンロードします。
- `pdf_to_csv.py` でセクション/文抽出を行うための正規表現 + PDF抽出パイプラインを備えています。
- レベル、セクション、ページ、文の確認が可能な抽出ユーティリティを `language_extraction.py` に実装しています。
- キャッシュ参照、プロンプト処理、基本的なJSON再試行を含むOpenAIリクエストレイヤーを `openai_request.py` で提供します。
- `app.py` と `templates/index.html` によって多言語レンダリングパイプラインを提供します。
- 生成した翻訳リクエストとレスポンスは `cache/` 配下のディスク上キャッシュで再利用します。

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
│   └── README.*.md
└── *.ipynb
```

## ✅ 前提条件

- Python `3.10+`
- `pip` と有効な仮想環境（`venv` 推奨）
- AI利用時のOpenAI APIキー（`OPENAI_API_KEY`）
- PDFダウンロードとOpenAIリクエストのための通信環境

このリポジトリにはロックファイルがないため、依存関係は import 文と既存内容から推定されます。

| パッケージ | 利用箇所 |
|---|---|
| `tornado` | `app.py` |
| `openai` | `openai_request.py`、`multilingual_sentence.py` |
| `PyPDF2` | `pdf_to_csv.py`、`language_extraction.py` |
| `requests` | `download_course_text.py` |
| `beautifulsoup4` | `download_course_text.py` |

## 🛠️ インストール

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

| セットアップヒント | コマンド |
|---|---|
| 仮想環境を有効化 | `source .venv/bin/activate` |
| 再現可能な環境を構築 | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| チェックを実行 | `python -m pip check` |

## 🚀 使い方

標準パイプラインは次の順序で実行します。

### 1) 元のPDFをダウンロード

```bash
python download_course_text.py
```

`downloaded_pdfs/` にPDFを保存します。

### 2) PDFの内容をCSVへ変換

```bash
python pdf_to_csv.py
```

既定で `japanese_language_data.csv` を出力します。

### 3) 特定のPDFスライスを確認（任意）

```bash
python language_extraction.py
```

広域データを生成する前に、特定の `level`、`section`、`page`、`sentence_num` の経路を検証するのに有用です。

### 4) 多言語文ペイロードを生成（任意）

```bash
python multilingual_sentence.py
```

信頼性に関する現在の挙動:

- 現在のバージョンは最初の1行のみを処理します（早期 `break` によるものです）。
- プロンプト生成では `japanese_text` を参照しており、抽出済みCSV行の変数名と不整合の可能性があり、失敗する場合があります。

### 5) Webアプリを起動

```bash
python app.py
```

- デフォルトのTornadoポート: `7788`
- URL: `http://localhost:7788/`
- ログの既知の不一致: 起動時の表示メッセージは現在 `http://localhost:8888` を参照します。

## ⚙️ 設定

実行時スクリプトが参照する環境変数:

| 変数 | 必須 | 目的 | デフォルト |
|---|---|---|---|
| `OPENAI_API_KEY` | はい（AIフローのみ） | OpenAI認証 | N/A |
| `OPENAI_MODEL` | いいえ | リクエスト時のモデル上書き | `gpt-4-0125-preview` |

実行時のファイル/ディレクトリ:

- `downloaded_pdfs/` — `download_course_text.py` が作成します。
- `cache/` — OpenAI のリクエスト/レスポンスペイロードをキャッシュします。
- `translations.json` — Tornado UIで使用されます。
- `templates/index.html` — ブラウザー表示テンプレート。

前提:

- すべてのスクリプトはリポジトリのルートディレクトリを作業ディレクトリとして想定しています。
- 翻訳キャッシュは、古い場合や欠落している場合に安全に再生成できます。

## 🧾 例

### CSV形式（`japanese_language_data.csv`）

```csv
Level,Unit,Section,Sentence No.,Content
```

### OpenAI翻訳ペイロード構造（`translations.json`）

```json
{
  "ja": {
    "full": "...",
    "pairs": [
      {
        "part": "日",
        "phonetic": "ひ"
      }
    ]
  },
  "en": { "full": "...", "pairs": [] },
  "ar": { "full": "...", "pairs": [] },
  "zh": { "full": "...", "pairs": [] },
  "yue": { "full": "...", "pairs": [] }
}
```

### 最小検証

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 開発ノート

- このプロジェクトはパッケージ化されていません（`requirements.txt`、`pyproject.toml`、CIは未導入）。
- スクリプト中心で、反復中に直接編集して再実行する運用を想定しています。
- ノートブックは探索用途で、製品版パイプラインというより調査支援として扱うべきです。
- `i18n/README.*.md` は既に存在し、このREADMEの上部言語ナビを共通エントリポイントとして使っています。

## 🩺 トラブルシューティング

- `ModuleNotFoundError`: 有効な仮想環境に必要なパッケージをすべてインストールしてください。
- `OPENAI` 認証エラー / 応答空レスポンス: `OPENAI_API_KEY` がシェルでエクスポートされているか確認します。
- `downloaded_pdfs` の `FileNotFoundError`: 先に `python download_course_text.py` を実行してください。
- OpenAI変換の問題: `cache/*.json` を確認し、`multilingual_sentence.py` が想定する入力ペイロード形式を検証します。
- アプリURLの混乱: 起動後に `http://localhost:7788/` にアクセスしてください。

## 🛣️ ロードマップ

- 再現可能なインストールのため、依存マニフェスト（`requirements.txt` または `pyproject.toml`）を追加します。
- `multilingual_sentence.py` の1行 `break` を削除し、フルバッチの多言語生成をサポートします。
- `multilingual_sentence.py` のプロンプト変数参照を修正し、出力検証を追加します。
- Tornadoの起動URLログを実際のポート `7788` に合わせて修正します。
- CLIフラグ（言語、レベル、ソースパス、出力パス）を追加します。
- 抽出、再試行/再解析ロジック、JSONスキーマ検証の軽量テストを導入します。
- `i18n` の言語別ドキュメントを拡充します。

## 🤝 コントリビュート

コントリビューション歓迎です。

1. リポジトリをフォークします。
2. フィーチャーブランチを作成します。
3. 変更は絞り、再現しやすいスクリプトワークフローを維持します。
4. 明確な理由を示したプルリクエストを提出します。

抽出ロジックを更新する場合は、PR説明に入出力サンプルを添えてください。

## 🙏 謝辞

- `rs_html.py` のRosetta Stoneコース教材リンクは、ダウンロード対象PDFコーパス参照元です。
- OpenAI API は多言語生成と音韻注釈実験に使用されています。



## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
