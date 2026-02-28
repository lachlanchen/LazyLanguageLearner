[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# lazylanguagelearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

用懒人方式学语言。

## 🌍 概览

LazyLanguageLearner 是一个基于 Python 的语言学习工作流，结合了：

- 为 Rosetta Stone 课程内容文档获取 PDF。
- 将 PDF 解析并抽取句子，生成 CSV 数据集。
- 基于 OpenAI 的多语言句子转换、带读音配对（phonetic pairs）以及本地磁盘缓存。
- 轻量级 Tornado Web 应用，用于渲染带 ruby 读音注释的多语言文本。

当前仓库以脚本驱动为主（尚未打包为 pip 模块），并且数据文件与笔记本直接包含在仓库中。

## ✨ 功能

- 从 `rs_html.py` 中嵌入的链接下载语言课程 PDF（`download_course_text.py`）。
- 将 PDF 中的章节/句子数据提取为结构化 CSV（`pdf_to_csv.py`、`language_extraction.py`）。
- 将 OpenAI 提示词/响应数据缓存到 `cache/*.json`，减少重复 API 调用（`openai_request.py`）。
- 将 AI 响应解析为 JSON，并带有重试逻辑与自定义 JSON 解析错误处理。
- 通过 Tornado 从 `translations.json` 提供多语言句子块（`app.py` + `templates/index.html`）。
- 在渲染前包含日语读音标准化（`katakana` 转 `hiragana`）。

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
│   └── (currently empty)
└── *.ipynb
```

## ✅ 前置要求

基于当前情况的假设（因为目前仓库中没有提交 lockfile 或依赖清单）：

- Python 3.10+（邻近版本大概率可用；但未声明精确测试矩阵）。
- `pip` 与 `venv`。
- 需要 OpenAI API Key 才能运行依赖模型的脚本。

根据 import 推断的 Python 依赖：

| Package | Used by |
|---|---|
| `tornado` | `app.py` 中的 Web 服务器 |
| `openai` | `openai_request.py` 中的 API 调用 |
| `PyPDF2` | 提取脚本中的 PDF 解析 |
| `requests` | `download_course_text.py` 中的 PDF 下载 |
| `beautifulsoup4` | 下载器中的 HTML 解析 |

## 🛠️ 安装

```bash
# from repository root
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

## 🚀 使用方法

### 1) 下载源 PDF

```bash
python download_course_text.py
```

这会创建 `downloaded_pdfs/`，并把语言/单元 PDF 保存到该目录。

### 2) 将日语 PDF 内容提取到 CSV

```bash
python pdf_to_csv.py
```

当前脚本默认输出：`japanese_language_data.csv`。

### 3) （可选）交互式切分章节/页码/句子文本

```bash
python language_extraction.py
```

脚本中包含可编辑的示例变量（`level`、`section`、`sentence_num`），并会打印提取结果。

### 4) 使用 OpenAI 流程生成多语言 JSON

```bash
python multilingual_sentence.py
```

当前行为说明：

- 只处理 CSV 的第一行（循环中有 `break`）。
- 脚本在构造提示词时当前引用了未定义变量（`japanese_text`），因此在稳定使用前需要一个小修复。

### 5) 运行 Web 应用

```bash
python app.py
```

- Tornado 监听端口 `7788`。
- 在浏览器中打开：`http://localhost:7788/`。
- 注意：尽管实际绑定的是 `7788`，当前启动日志会打印 `http://localhost:8888`。

## ⚙️ 配置

环境变量：

| Variable | Required | Purpose | Current default |
|---|---|---|---|
| `OPENAI_API_KEY` | Yes | `OpenAI()` 客户端初始化必需 | N/A |
| `OPENAI_MODEL` | No | 可选覆盖聊天模型 | `gpt-4-0125-preview` |

运行时文件/目录：

- `downloaded_pdfs/`：由下载脚本创建，供提取脚本使用。
- `cache/`：OpenAI 调用的请求/响应缓存。
- `translations.json`：Tornado UI 渲染的数据源。

## 🧾 数据格式示例

### CSV (`japanese_language_data.csv`)

`pdf_to_csv.py` 使用的表头：

```csv
Level,Unit,Section,Sentence No.,Content
```

### JSON (`translations.json`)

Web UI 期望语言键中包含 `pairs` 项，每项含有 `part` 与 `phonetic`：

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

## 🧪 开发说明

- 当前仓库没有 `requirements.txt`、`pyproject.toml` 或 CI 工作流。
- 脚本设计为从仓库根目录直接执行。
- 现有笔记本（`*.ipynb`）看起来更偏探索/原型用途。
- 较大的 CSV 产物直接纳入 Git 版本管理。
- `i18n/` 已存在，可用于放置翻译版 README。

## 🩺 故障排查

- `ModuleNotFoundError`：在当前虚拟环境安装上述推断依赖。
- `OPENAI` 认证错误：确认已在 shell 中导出 `OPENAI_API_KEY`。
- `FileNotFoundError: downloaded_pdfs`：先运行 `python download_course_text.py`。
- `multilingual_sentence.py` 在 `japanese_text` 处失败：把提示词构造中的 `japanese_text` 替换为 `content`。
- 端口混淆：除非修改 `app.listen(...)`，否则使用 `http://localhost:7788/`。

## 🛣️ 路线图

项目的潜在下一步：

- 添加依赖清单（`requirements.txt` 或 `pyproject.toml`）。
- 修复 `multilingual_sentence.py` 的提示词变量，并移除仅处理一行的 `break` 以支持批量处理。
- 让 Tornado 启动打印 URL 与实际绑定端口一致。
- 为 PDF 提取正则行为与 JSON 解析/重试逻辑补充测试。
- 增加语言/级别/路径等 CLI 参数，减少对脚本内变量手动编辑。
- 在 `i18n/` 中补齐更多翻译版 README 文件。

## 🤝 贡献

欢迎贡献。

1. Fork 本仓库。
2. 创建功能分支。
3. 进行聚焦变更并使用清晰的提交信息。
4. 提交 Pull Request，说明改了什么以及为什么改。

如果你修改了提取逻辑，建议附上示例输入/输出片段，便于评审。

## 🙏 致谢

- Rosetta Stone 课程内容链接嵌入在 `rs_html.py` 中，并作为 PDF 下载的来源参考。
- OpenAI API 用于多语言生成与读音结构化。

## 📄 许可证

本项目基于 Apache License 2.0 许可。详见 [LICENSE](LICENSE)。
