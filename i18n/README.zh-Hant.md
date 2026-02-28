[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# lazylanguagelearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

用懶人的方式學語言。

## 🌍 概覽

LazyLanguageLearner 是一個以 Python 為基礎的語言學習工作流程，結合了：

- 取得 Rosetta Stone 課程內容文件的 PDF。
- 將 PDF 解析並擷取句子為 CSV 資料集。
- 使用 OpenAI 進行多語句子轉換、產生讀音配對（phonetic pairs），並提供本機磁碟快取。
- 輕量 Tornado Web 應用，渲染帶有 ruby 讀音註解的多語文字。

目前此儲存庫以腳本驅動為主（尚未打包成 pip 模組），資料檔與筆記本直接包含在 repo 內。

## ✨ 功能

- 從 `rs_html.py` 內嵌連結下載語言課程 PDF（`download_course_text.py`）。
- 將 PDF 的章節/句子資料擷取為結構化 CSV（`pdf_to_csv.py`、`language_extraction.py`）。
- 將 OpenAI prompt/response 資料快取到 `cache/*.json`，降低重複 API 使用（`openai_request.py`）。
- 將 AI 回應解析為 JSON，包含重試邏輯與自訂 JSON 解析錯誤。
- 透過 Tornado 從 `translations.json` 提供多語句子區塊（`app.py` + `templates/index.html`）。
- 在渲染前包含日文讀音正規化（`katakana` 轉 `hiragana`）。

## 🗂️ 專案結構

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

## ✅ 先決條件

前提假設（因目前尚未提交 lockfile 或相依清單）：

- Python 3.10+（鄰近版本可能可用；未宣告精確測試矩陣）。
- `pip` 與 `venv`。
- 需有 OpenAI API key 才能執行模型相關腳本。

從 import 推斷的 Python 相依套件：

| Package | Used by |
|---|---|
| `tornado` | `app.py` 中的 Web server |
| `openai` | `openai_request.py` 中的 API 呼叫 |
| `PyPDF2` | 擷取腳本中的 PDF 解析 |
| `requests` | `download_course_text.py` 中的 PDF 下載 |
| `beautifulsoup4` | 下載器中的 HTML 解析 |

## 🛠️ 安裝

```bash
# from repository root
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

## 🚀 使用方式

### 1) 下載來源 PDF

```bash
python download_course_text.py
```

此步驟會建立 `downloaded_pdfs/`，並將語言/單元 PDF 儲存至該目錄。

### 2) 將日文 PDF 內容擷取到 CSV

```bash
python pdf_to_csv.py
```

目前腳本的預設輸出：`japanese_language_data.csv`。

### 3) （可選）互動式切分章節/頁面/句子文字

```bash
python language_extraction.py
```

腳本內含可編輯範例變數（`level`、`section`、`sentence_num`），並會印出擷取文字。

### 4) 使用 OpenAI 流程產生多語 JSON

```bash
python multilingual_sentence.py
```

目前行為說明：

- 僅處理 CSV 第一列（迴圈中有 `break`）。
- 腳本目前在建立 prompt 時參照了未定義變數（`japanese_text`），在可靠使用前需要小幅修正。

### 5) 執行 Web 應用

```bash
python app.py
```

- Tornado 監聽埠 `7788`。
- 瀏覽器開啟：`http://localhost:7788/`。
- 注意：啟動日誌目前會印出 `http://localhost:8888`，但實際綁定是 `7788`。

## ⚙️ 設定

環境變數：

| Variable | Required | Purpose | Current default |
|---|---|---|---|
| `OPENAI_API_KEY` | Yes | `OpenAI()` client 初始化必需 | N/A |
| `OPENAI_MODEL` | No | 可選覆寫 chat model | `gpt-4-0125-preview` |

執行期檔案/目錄：

- `downloaded_pdfs/`：由下載器建立，供擷取腳本使用。
- `cache/`：OpenAI 呼叫的 request/response 快取。
- `translations.json`：Tornado UI 渲染資料來源。

## 🧾 資料格式範例

### CSV (`japanese_language_data.csv`)

`pdf_to_csv.py` 使用的標頭：

```csv
Level,Unit,Section,Sentence No.,Content
```

### JSON (`translations.json`)

Web UI 預期語言鍵包含 `pairs` 項目，每項帶有 `part` 與 `phonetic`：

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

## 🧪 開發備註

- 此 repo 目前沒有 `requirements.txt`、`pyproject.toml` 或 CI workflow。
- 腳本設計為直接從 repository root 執行。
- 既有筆記本（`*.ipynb`）看起來偏向探索/原型用途。
- 大型 CSV 產物直接版本控管於 Git。
- `i18n/` 已存在，可用於放置翻譯版 README。

## 🩺 疑難排解

- `ModuleNotFoundError`：在啟用中的虛擬環境安裝推斷相依套件。
- `OPENAI` 驗證錯誤：確認已在 shell 匯出 `OPENAI_API_KEY`。
- `FileNotFoundError: downloaded_pdfs`：先執行 `python download_course_text.py`。
- `multilingual_sentence.py` 在 `japanese_text` 失敗：將 prompt 建構中的 `japanese_text` 改為 `content`。
- 埠號混淆：除非你修改 `app.listen(...)`，否則使用 `http://localhost:7788/`。

## 🛣️ 路線圖

專案可能的下一步：

- 新增相依清單（`requirements.txt` 或 `pyproject.toml`）。
- 修正 `multilingual_sentence.py` 的 prompt 變數，並移除僅一列處理的 `break` 以支援批次。
- 讓 Tornado 啟動時印出的 URL 與實際綁定埠一致。
- 為 PDF 擷取正則行為與 JSON 解析/重試邏輯新增測試。
- 新增語言/等級/路徑的 CLI 參數，減少檔內手動編輯。
- 在 `i18n/` 補齊更多翻譯版 README 檔。

## 🤝 貢獻

歡迎貢獻。

1. Fork 此儲存庫。
2. 建立功能分支。
3. 進行聚焦變更並撰寫清楚的 commit 訊息。
4. 開啟 pull request，說明變更內容與原因。

若你修改擷取邏輯，請附上範例輸入/輸出片段，讓審查更容易。

## 🙏 致謝

- Rosetta Stone 課程內容連結內嵌於 `rs_html.py`，並作為 PDF 下載來源參考。
- OpenAI API 用於多語生成與讀音結構化。

## 📄 授權

本專案採用 Apache License 2.0。詳見 [LICENSE](LICENSE)。
