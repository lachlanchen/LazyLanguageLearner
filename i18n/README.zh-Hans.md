[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner — apps and four buddies](https://lazying.art/media/brand/lazyingart-intro.gif)](https://lazying.art)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| 属性 | 值 |
|---|---|
| 类型 | 脚本驱动的多语言语言学习流水线 |
| 运行时 | Python CLI + Tornado Web 应用 |
| 主要来源 | Rosetta Stone 课程 PDF |
| 存储 | 本地 CSV + JSON 缓存文件 |
| 默认端口 | `7788` |

LazyLanguageLearner 是一个脚本驱动的 Python 工作流，用于将语言课程 PDF 转换为可复用的多语言学习数据，并在一个简洁的 Web UI 中渲染。

## 🌍 概览

该仓库将内容提取、转换和服务化能力整合在一起：

| 步骤 | 目的 |
|---|---|
| 1 | 从 `rs_html.py` 中嵌入的链接下载 Rosetta Stone 课程内容 PDF。 |
| 2 | 将 PDF 解析为句子级别的 CSV 行，以便后续转换。 |
| 3 | 通过 OpenAI 生成多语言/音标变体，并使用本地磁盘缓存。 |
| 4 | 在 Tornado Web UI 中渲染带有音标标注的结构化句子。 |

本项目刻意保持轻量且以仓库根目录为中心：脚本设计为直接在仓库根目录运行，而不是作为已安装包运行。

## ✨ 功能特性

- 使用 `download_course_text.py`，基于 `rs_html.py` 中的内嵌链接实现**自动化下载流程**。
- `pdf_to_csv.py` 中的**正则 + PDF 提取流水线**用于章节和句子的提取。
- `language_extraction.py` 提供按级别、章节、页码与句号进行的**选择性提取工具**。
- `openai_request.py` 提供 OpenAI 请求层，支持缓存查询、提示词处理以及基础 JSON 提取重试。
- `app.py` 与 `templates/index.html` 提供跨语言渲染流水线。
- **日语音标归一化**：在渲染前将片假名数据转换为平假名。
- 在 `cache/` 下为生成后的翻译请求与响应提供**磁盘缓存**。

## 🗂️ 项目结构

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

## ✅ 先决条件

- Python `3.10+`
- 带有有效虚拟环境的 `pip`（推荐使用 `venv`）
- 使用 AI 生成功能时需提供 OpenAI API Key（`OPENAI_API_KEY`）
- 进行 PDF 下载与 OpenAI 请求时需要可用的网络连接

由于仓库中没有锁文件，依赖项依据导入和历史内容推断：

| 包 | 用途 |
|---|---|
| `tornado` | `app.py` |
| `openai` | `openai_request.py`, `multilingual_sentence.py` |
| `PyPDF2` | `pdf_to_csv.py`, `language_extraction.py` |
| `requests` | `download_course_text.py` |
| `beautifulsoup4` | `download_course_text.py` |

## 🛠️ 安装

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

| 设置提示 | 命令 |
|---|---|
| 激活虚拟环境 | `source .venv/bin/activate` |
| 重建环境 | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| 运行检查 | `python -m pip check` |

## 🚀 使用方法

按如下顺序运行脚本完成标准流水线：

### 1) 下载源 PDF

```bash
python download_course_text.py
```

PDF 会下载到 `downloaded_pdfs/`。

### 2) 提取 PDF 内容为 CSV

```bash
python pdf_to_csv.py
```

默认会生成 `japanese_language_data.csv`。

### 3) 检查特定 PDF 片段（可选）

```bash
python language_extraction.py
```

在生成更大规模数据前，可用于验证特定的 `level`、`section`、`page`、`sentence_num` 路径。

### 4) 构建多语言句子载荷（可选）

```bash
python multilingual_sentence.py
```

当前行为说明（为保证可靠性）：

- 受早期 `break` 语句影响，当前版本只处理第一行。
- 提示词生成中引用了 `japanese_text`，该变量与当前提取的 CSV 行变量存在不一致，可能会导致失败。

### 5) 启动 Web 应用

```bash
python app.py
```

- 默认 Tornado 端口：`7788`
- 地址：`http://localhost:7788/`
- 日志中需确认的已知不一致：启动打印信息当前仍显示 `http://localhost:8888`。

## ⚙️ 配置

运行时脚本期望的环境变量：

| 变量 | 是否必需 | 用途 | 默认值 |
|---|---|---|---|
| `OPENAI_API_KEY` | 是（仅 AI 流程） | OpenAI 认证 | N/A |
| `OPENAI_MODEL` | 否 | 请求中覆盖模型 | `gpt-4-0125-preview` |

运行文件/目录：

- `downloaded_pdfs/` — 由 `download_course_text.py` 填充。
- `cache/` — 存储 OpenAI 提示词与响应缓存。
- `translations.json` — 供 Tornado UI 使用。
- `templates/index.html` — 浏览器渲染模板。

假设条件：

- 仓库根目录是所有脚本的预期工作目录。
- 语言缓存可在过期或缺失时安全重建。

## 🧾 示例

### CSV 格式（`japanese_language_data.csv`）

```csv
Level,Unit,Section,Sentence No.,Content
```

### OpenAI 翻译载荷结构（`translations.json`）

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

### 最小化快速检查

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 开发说明

- 本项目未打包（未包含 `requirements.txt`、`pyproject.toml`，也未配置 CI）。
- 脚本以第一优先级设计，适合在迭代过程中直接编辑和重跑。
- Notebook 文件偏向探索用途，应视为研究辅助，而非生产流水线。
- `i18n/README.*.md` 已存在多语言文档，本文件顶部的语言导航作为统一入口。

## 🩺 故障排查

- `ModuleNotFoundError`：在当前激活的虚拟环境中安装所有必需包。
- `OPENAI` 认证错误 / 空响应：确认已在 Shell 中导出 `OPENAI_API_KEY`。
- `downloaded_pdfs` 报 `FileNotFoundError`：先运行 `python download_course_text.py`。
- OpenAI 转换问题：检查 `cache/*.json`，并确认 `multilingual_sentence.py` 期望的输入载荷格式。
- 应用地址困惑：启动后访问 `http://localhost:7788/`。

## 🛣️ 路线图

- 增加依赖清单（`requirements.txt` 或 `pyproject.toml`）以提高可重现性。
- 去掉 `multilingual_sentence.py` 中仅处理一行的 `break`，支持完整批量多语言生成。
- 修复 `multilingual_sentence.py` 中提示词变量的使用问题，并补充输出校验。
- 修正 Tornado 启动时日志中的 URL，确保与端口 `7788` 一致。
- 增加 CLI 参数（语言、级别、源路径、输出路径）。
- 引入轻量级测试，覆盖提取逻辑、重试/解析逻辑和 JSON schema 校验。
- 扩展 `i18n` 中面向贡献者的文档。

## 🤝 贡献指南

欢迎提交贡献。

1. Fork 本仓库。
2. 创建功能分支。
3. 做出聚焦变更，并保持脚本流程可复现。
4. 提交 PR，附上清晰的变更理由及变更前后行为说明。

如果你更新了提取逻辑，请在 PR 描述中补充示例输入与输出。

## 🙏 致谢

- `rs_html.py` 中的 Rosetta Stone 课程内容链接提供了可下载 PDF 语料来源。
- OpenAI API 被用于多语言生成和音标标注实验。



## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
