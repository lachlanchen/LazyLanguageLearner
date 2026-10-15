[English](README.md) · [العربية](i18n/README.ar.md) · [Español](i18n/README.es.md) · [Français](i18n/README.fr.md) · [日本語](i18n/README.ja.md) · [한국어](i18n/README.ko.md) · [Tiếng Việt](i18n/README.vi.md) · [中文 (简体)](i18n/README.zh-Hans.md) · [中文（繁體）](i18n/README.zh-Hant.md) · [Deutsch](i18n/README.de.md) · [Русский](i18n/README.ru.md)


[![LazyingArt banner — apps and four buddies](https://lazying.art/media/brand/lazyingart-intro.gif)](https://lazying.art)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| Attribute | Value |
|---|---|
| Type | Script-driven multilingual language learning pipeline |
| Runtime | Python CLI + Tornado web app |
| Main source | Rosetta Stone course PDFs |
| Storage | Local CSV + JSON cache files |
| Default port | `7788` |

LazyLanguageLearner is a script-driven Python workflow for turning language-course PDFs into reusable multilingual learning data and rendering it in a minimal web UI.

## 🌍 Overview

The repository combines content extraction, transformation, and serving:

| Step | Purpose |
|---|---|
| 1 | Download Rosetta Stone course-content PDFs from links embedded in `rs_html.py`. |
| 2 | Parse PDFs into sentence-level CSV rows for transformation. |
| 3 | Generate multilingual/phonetic variants through OpenAI with on-disk caching. |
| 4 | Render the structured sentences in a Tornado web UI with phonetic annotations. |

This project is intentionally lightweight and root-centric: scripts are designed to be run directly from the repository root rather than as an installed package.

## ✨ Features

- **Automated download workflow** from embedded links in `rs_html.py` using `download_course_text.py`.
- **Regex + PDF extraction pipeline** for section/sentence extraction in `pdf_to_csv.py`.
- **Selective extraction utilities** for level, section, page, and sentence inspection in `language_extraction.py`.
- **OpenAI request layer** (`openai_request.py`) with cache lookups, prompt handling, and basic JSON extraction retries.
- **Cross-language rendering pipeline** served by `app.py` and `templates/index.html`.
- **Japanese phonetic normalization** converting katakana data to hiragana before rendering.
- **On-disk caching** under `cache/` for generated translation requests and responses.

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
│   └── README.*.md
└── *.ipynb
```

## ✅ Prerequisites

- Python `3.10+`
- `pip` with an active virtual environment (`venv` recommended)
- An OpenAI API key (`OPENAI_API_KEY`) when using AI-powered generation
- A working internet connection for PDF downloads and OpenAI requests

Because there is no lockfile in this repo, dependencies are inferred from imports and previous content:

| Package | Used by |
|---|---|
| `tornado` | `app.py` |
| `openai` | `openai_request.py`, `multilingual_sentence.py` |
| `PyPDF2` | `pdf_to_csv.py`, `language_extraction.py` |
| `requests` | `download_course_text.py` |
| `beautifulsoup4` | `download_course_text.py` |

## 🛠️ Installation

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

| Setup hint | Command |
|---|---|
| Activate venv | `source .venv/bin/activate` |
| Reproduce env | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| Run checks | `python -m pip check` |

## 🚀 Usage

Run scripts in this order for the standard pipeline:

### 1) Download source PDFs

```bash
python download_course_text.py
```

Downloads PDFs into `downloaded_pdfs/`.

### 2) Extract PDF content to CSV

```bash
python pdf_to_csv.py
```

Outputs `japanese_language_data.csv` by default.

### 3) Inspect a specific PDF slice (optional)

```bash
python language_extraction.py
```

Useful for validating specific `level`, `section`, `page`, and `sentence_num` paths before generating broader datasets.

### 4) Build multilingual sentence payloads (optional)

```bash
python multilingual_sentence.py
```

Current behavior notes for reliability:

- The current version processes only the first row due to an early `break`.
- Prompt generation references `japanese_text`, which currently appears inconsistent with the extracted CSV row variable and may fail.

### 5) Start the web app

```bash
python app.py
```

- Default Tornado port: `7788`
- URL: `http://localhost:7788/`
- Known mismatch to verify in logs: the startup print message currently references `http://localhost:8888`.

## ⚙️ Configuration

Environment variables expected by the runtime scripts:

| Variable | Required | Purpose | Default |
|---|---|---|---|
| `OPENAI_API_KEY` | Yes (AI flow only) | OpenAI authentication | N/A |
| `OPENAI_MODEL` | No | Model override in requests | `gpt-4-0125-preview` |

Runtime files/directories:

- `downloaded_pdfs/` — populated by `download_course_text.py`.
- `cache/` — stores cached OpenAI prompt/response payloads.
- `translations.json` — consumed by the Tornado UI.
- `templates/index.html` — browser rendering template.

Assumptions:

- The repository root is the intended working directory for all scripts.
- Translation cache can be regenerated safely if stale or missing.

## 🧾 Examples

### CSV format (`japanese_language_data.csv`)

```csv
Level,Unit,Section,Sentence No.,Content
```

### OpenAI translation payload shape (`translations.json`)

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

### Minimal quick check

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 Development Notes

- The project is not packaged (`requirements.txt`, `pyproject.toml`, and CI not present).
- Scripts are script-first and meant to be edited and rerun during iteration.
- Notebook files appear exploratory and should be treated as research aids, not production pipelines.
- `i18n/README.*.md` already exists for multilingual documentation, with the top-level language nav block in this file acting as the shared entry point.

## 🩺 Troubleshooting

- `ModuleNotFoundError`: install all required packages in the active virtual environment.
- `OPENAI` auth error / empty responses: check that `OPENAI_API_KEY` is exported in your shell.
- `FileNotFoundError` for `downloaded_pdfs`: run `python download_course_text.py` first.
- OpenAI conversion issues: inspect `cache/*.json` and verify input payload format expected by `multilingual_sentence.py`.
- App URL confusion: browse `http://localhost:7788/` after launch.

## 🛣️ Roadmap

- Add a dependency manifest (`requirements.txt` or `pyproject.toml`) for reproducible installs.
- Remove the one-row `break` from `multilingual_sentence.py` and support full-batch multilingual generation.
- Fix prompt variable usage in `multilingual_sentence.py` and add output validation.
- Correct Tornado startup URL log to reflect port `7788`.
- Add CLI flags (language, level, source paths, output path).
- Introduce lightweight tests for extraction, retry/parse logic, and JSON schema validation.
- Expand contributor-facing docs in `i18n` language variants.

## 🤝 Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Make a focused change and keep script workflows easy to reproduce.
4. Open a pull request with clear rationale and before/after behavior notes.

If you update extraction logic, include sample inputs and outputs in your PR description.

## 🙏 Acknowledgements

- Rosetta Stone course-content links in `rs_html.py` are the source of downloadable PDF corpus references.
- OpenAI APIs are used for multilingual generation and phonetic annotation experiments.

## ❤️ Support

<!-- support:start -->
[![GitHub Sponsors](https://img.shields.io/badge/GitHub-Sponsors-EA4AAA?style=for-the-badge&logo=githubsponsors&logoColor=white)](https://github.com/sponsors/lachlanchen)
[![LazyingArt Support](https://img.shields.io/badge/LazyingArt-Support-0EA5E9?style=for-the-badge)](https://chat.lazying.art/donate)
[![PayPal Support](https://img.shields.io/badge/PayPal-Support-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen)
[![Stripe Support](https://img.shields.io/badge/Stripe-Support-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400)
<!-- support:end -->

## 📄 License

This project is licensed under the Apache License 2.0. See [LICENSE](LICENSE).
