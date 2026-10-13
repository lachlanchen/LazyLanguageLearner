[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner — apps and four buddies](https://github.com/lachlanchen/LazyingArtWebsite/raw/main/media/brand/lazyingart-intro.gif)](https://lazying.art)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| 屬性 | 值 |
|---|---|
| 類型 | 以腳本為核心的多語言學習流程 |
| 執行環境 | Python CLI + Tornado 網頁應用 |
| 主要來源 | Rosetta Stone 課程 PDF |
| 儲存方式 | 本機 CSV + JSON 快取檔案 |
| 預設埠號 | `7788` |

LazyLanguageLearner 是一個以腳本為核心的 Python 工作流程，將語言課程 PDF 轉成可重複使用的多語學習資料，並透過精簡的網頁介面呈現。

## 🌍 概覽

本專案整合了內容提取、轉換與服務輸出：

| 步驟 | 目的 |
|---|---|
| 1 | 從 `rs_html.py` 內嵌的連結下載 Rosetta Stone 的課程 PDF。 |
| 2 | 將 PDF 解析為逐句 CSV 列，供後續轉換使用。 |
| 3 | 透過 OpenAI 產生多語言／音標變體並快取到本機。 |
| 4 | 以 Tornado 網頁介面渲染結構化句子並附帶音標註記。 |

此專案刻意保持輕量且以專案根目錄為主：腳本設計上應直接在 repo 根目錄執行，而非作為已安裝套件。

## ✨ 功能

- **自動下載流程**：使用 `download_course_text.py` 從 `rs_html.py` 的內嵌連結下載課程內容。
- **Regex + PDF 提取管線**：`pdf_to_csv.py` 提供段落與句子擷取能力。
- **選擇式提取工具**：`language_extraction.py` 支援等級、章節、頁碼與句子檢查。
- **OpenAI 請求層** (`openai_request.py`)：含快取查詢、提示詞處理與基本 JSON 重試擷取。
- **跨語言渲染管線**：由 `app.py` 與 `templates/index.html` 服務。
- **日語音標正規化**：在渲染前將片假名資料轉換為平假名。
- **本機快取**：`cache/` 目錄儲存已生成的翻譯請求與回應。

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
│   └── README.*.md
└── *.ipynb
```

## ✅ 先決條件

- Python `3.10+`
- `pip` 與啟用中的虛擬環境（建議使用 `venv`）
- 使用 AI 生成功能時需提供 OpenAI API 金鑰（`OPENAI_API_KEY`）
- 下載 PDF 與 OpenAI 請求時需要可用的網路連線

本專案沒有 lockfile，故依照匯入模組與既有內容推斷相依套件：

| 套件 | 用途 |
|---|---|
| `tornado` | `app.py` |
| `openai` | `openai_request.py`、`multilingual_sentence.py` |
| `PyPDF2` | `pdf_to_csv.py`、`language_extraction.py` |
| `requests` | `download_course_text.py` |
| `beautifulsoup4` | `download_course_text.py` |

## 🛠️ 安裝

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

| 設定提示 | 指令 |
|---|---|
| 啟用虛擬環境 | `source .venv/bin/activate` |
| 重建環境 | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| 執行檢查 | `python -m pip check` |

## 🚀 使用方式

請依下列順序執行腳本以完成標準流程：

### 1) 下載原始 PDF

```bash
python download_course_text.py
```

將 PDF 下載到 `downloaded_pdfs/`。

### 2) 將 PDF 轉為 CSV

```bash
python pdf_to_csv.py
```

預設輸出 `japanese_language_data.csv`。

### 3) 檢視特定 PDF 片段（可選）

```bash
python language_extraction.py
```

在進行大規模資料生成前，用以驗證指定的 `level`、`section`、`page` 與 `sentence_num`。

### 4) 建立多語句子 payload（可選）

```bash
python multilingual_sentence.py
```

目前的穩定性備註：

- 目前版本因提前 `break` 的關係僅會處理第一列資料。
- 提示詞產生時引用 `japanese_text`，目前與已解析 CSV 的列變數不一致，可能導致失敗。

### 5) 啟動網頁應用

```bash
python app.py
```

- 預設 Tornado 連線埠：`7788`
- URL：`http://localhost:7788/`
- 記錄輸出中的已知不一致：啟動訊息目前仍會顯示 `http://localhost:8888`，請以實際啟動為準。

## ⚙️ 設定

執行腳本時預期的環境變數：

| 變數 | 必要 | 用途 | 預設值 |
|---|---|---|---|
| `OPENAI_API_KEY` | 是（僅 AI 流程） | OpenAI 驗證 | N/A |
| `OPENAI_MODEL` | 否 | 覆寫請求中的模型 | `gpt-4-0125-preview` |

執行時的檔案與資料夾：

- `downloaded_pdfs/`：由 `download_course_text.py` 建立。
- `cache/`：儲存 OpenAI 請求／回應快取。
- `translations.json`：由 Tornado UI 使用。
- `templates/index.html`：瀏覽器端渲染模板。

前提條件：

- 專案根目錄是所有腳本的預期工作目錄。
- 翻譯快取若遺失或過期，建議安全地重建。

## 🧾 範例

### CSV 格式（`japanese_language_data.csv`）

```csv
Level,Unit,Section,Sentence No.,Content
```

### OpenAI 翻譯 payload 結構（`translations.json`）

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

### 最小快速檢查

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 開發備註

- 本專案未打包（未提供 `requirements.txt`、`pyproject.toml`，亦未配置 CI）。
- 腳本是第一優先，應在迭代過程中直接編輯並重跑。
- Notebook 檔案多為探索用途，應僅作為研究輔助，不建議作為正式管線。
- `i18n/README.*.md` 目前已存在多語文件，且本檔案最上方的語系導覽列作為共用入口。

## 🩺 疑難排解

- `ModuleNotFoundError`：請先在啟用的虛擬環境中安裝所有必要套件。
- `OPENAI` 認證錯誤／空回應：請確認 `OPENAI_API_KEY` 已在 shell 中輸出為環境變數。
- `downloaded_pdfs` 找不到：先執行 `python download_course_text.py`。
- OpenAI 轉換問題：檢查 `cache/*.json`，並確認輸入 payload 格式符合 `multilingual_sentence.py` 的預期。
- 網址混淆：啟動後請以 `http://localhost:7788/` 存取。

## 🛣️ 路線圖

- 新增相依套件清單（`requirements.txt` 或 `pyproject.toml`）以讓安裝可重現。
- 移除 `multilingual_sentence.py` 中只處理一列的 `break`，並支援完整批次多語生成。
- 修正 `multilingual_sentence.py` 的提示變數使用，並新增輸出驗證。
- 修正 Tornado 啟動 URL log，使其對應埠號 `7788`。
- 加入 CLI 參數（語言、等級、來源路徑、輸出路徑）。
- 增加輕量測試，覆蓋提取、重試/解析邏輯與 JSON schema 驗證。
- 擴充 `i18n` 多語文件、優化貢獻者導向說明。

## 🤝 貢獻指南

歡迎任何形式的貢獻。

1. Fork 本儲存庫。
2. 建立一個功能分支。
3. 聚焦修改內容並保持腳本流程可重現。
4. 開啟 Pull Request，並清楚說明改動原因與前後行為差異。

若你更新了抽取邏輯，請在 PR 說明中包含輸入/輸出範例。

## 🙏 致謝

- `rs_html.py` 中的 Rosetta Stone 課程內容連結提供可下載 PDF 的語料來源。
- OpenAI API 用於多語生成與音標註記實驗。



## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
