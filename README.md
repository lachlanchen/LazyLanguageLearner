[English](README.md) · [العربية](i18n/README.ar.md) · [Español](i18n/README.es.md) · [Français](i18n/README.fr.md) · [日本語](i18n/README.ja.md) · [한국어](i18n/README.ko.md) · [Tiếng Việt](i18n/README.vi.md) · [中文 (简体)](i18n/README.zh-Hans.md) · [中文（繁體）](i18n/README.zh-Hant.md) · [Deutsch](i18n/README.de.md) · [Русский](i18n/README.ru.md)

# lazylanguagelearner


[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

Learn language in a lazy way.

## 🌍 Overview

LazyLanguageLearner is a Python-based language-learning workflow that combines:

- PDF acquisition for Rosetta Stone course-content documents.
- PDF parsing and sentence extraction into CSV datasets.
- OpenAI-powered multilingual sentence conversion with phonetic pairs and local disk caching.
- A lightweight Tornado web app that renders multilingual text with ruby phonetic annotations.

The current repository is script-driven (not packaged as a pip module yet), with data files and notebooks included directly in the repo.

## ✨ Features

- Downloads language course PDFs from links embedded in `rs_html.py` (`download_course_text.py`).
- Extracts section/sentence data from PDFs into structured CSV (`pdf_to_csv.py`, `language_extraction.py`).
- Caches OpenAI prompt/response data into `cache/*.json` to reduce repeated API usage (`openai_request.py`).
- Parses AI responses into JSON with retry logic and custom JSON parsing errors.
- Serves multilingual sentence blocks from `translations.json` through Tornado (`app.py` + `templates/index.html`).
- Includes Japanese phonetic normalization (`katakana` to `hiragana`) before rendering.

## 🗂️ Project Structure

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

## ✅ Prerequisites

Assumptions (because no lockfile or dependency manifest is currently committed):

- Python 3.10+ (likely works on nearby versions; exact tested matrix is not declared).
- `pip` and `venv`.
- OpenAI API key for model-backed scripts.

Inferred Python dependencies from imports:

| Package | Used by |
|---|---|
| `tornado` | Web server in `app.py` |
| `openai` | API calls in `openai_request.py` |
| `PyPDF2` | PDF parsing in extraction scripts |
| `requests` | PDF downloads in `download_course_text.py` |
| `beautifulsoup4` | HTML parsing in downloader |

## 🛠️ Installation

```bash
# from repository root
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

## 🚀 Usage

### 1) Download source PDFs

```bash
python download_course_text.py
```

This creates `downloaded_pdfs/` and saves language/unit PDFs there.

### 2) Extract Japanese PDF content to CSV

```bash
python pdf_to_csv.py
```

Default output from current script: `japanese_language_data.csv`.

### 3) (Optional) Slice section/page/sentence text interactively

```bash
python language_extraction.py
```

The script contains editable example variables (`level`, `section`, `sentence_num`) and prints extracted text.

### 4) Generate multilingual JSON using OpenAI flow

```bash
python multilingual_sentence.py
```

Current behavior notes:

- Only first CSV row is processed (`break` in loop).
- Script currently references an undefined variable (`japanese_text`) in prompt creation, so it needs a small fix before reliable use.

### 5) Run the web app

```bash
python app.py
```

- Tornado listens on port `7788`.
- Open in browser: `http://localhost:7788/`.
- Note: startup log currently prints `http://localhost:8888` even though binding is `7788`.

## ⚙️ Configuration

Environment variables:

| Variable | Required | Purpose | Current default |
|---|---|---|---|
| `OPENAI_API_KEY` | Yes | Required by `OpenAI()` client initialization | N/A |
| `OPENAI_MODEL` | No | Optional override for chat model | `gpt-4-0125-preview` |

Runtime files/directories:

- `downloaded_pdfs/`: created by downloader, used by extraction scripts.
- `cache/`: request/response cache for OpenAI calls.
- `translations.json`: data source for Tornado UI rendering.

## 🧾 Data Format Examples

### CSV (`japanese_language_data.csv`)

Header used by `pdf_to_csv.py`:

```csv
Level,Unit,Section,Sentence No.,Content
```

### JSON (`translations.json`)

The web UI expects language keys with `pairs` entries containing `part` and `phonetic`:

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

## 🧪 Development Notes

- This repo currently has no `requirements.txt`, `pyproject.toml`, or CI workflow.
- Scripts are designed for direct execution from repository root.
- Existing notebooks (`*.ipynb`) appear exploratory/prototyping-oriented.
- Large CSV artifacts are versioned directly in Git.
- `i18n/` exists and is ready for translated README variants.

## 🩺 Troubleshooting

- `ModuleNotFoundError`: install inferred dependencies in the active virtual environment.
- `OPENAI` auth errors: ensure `OPENAI_API_KEY` is exported in the shell.
- `FileNotFoundError: downloaded_pdfs`: run `python download_course_text.py` first.
- `multilingual_sentence.py` failure on `japanese_text`: replace `japanese_text` with `content` in prompt construction.
- App port confusion: use `http://localhost:7788/` unless `app.listen(...)` is changed.

## 🛣️ Roadmap

Potential next steps for the project:

- Add dependency manifest (`requirements.txt` or `pyproject.toml`).
- Fix `multilingual_sentence.py` prompt variable and remove one-row `break` for batch processing.
- Align Tornado startup print URL with bound port.
- Add tests for PDF extraction regex behavior and JSON parsing/retry logic.
- Add CLI arguments for language/level/paths to reduce in-file edits.
- Populate `i18n/` with translated README files.

## 🤝 Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Make focused changes with clear commit messages.
4. Open a pull request describing what changed and why.

If you modify extraction logic, include sample input/output snippets to make review easier.

## 🙏 Acknowledgements

- Rosetta Stone course-content links are embedded in `rs_html.py` and used as source references for PDF downloads.
- OpenAI API is used for multilingual generation and phonetic structuring.

## 📄 License

This project is licensed under the Apache License 2.0. See [LICENSE](LICENSE).
