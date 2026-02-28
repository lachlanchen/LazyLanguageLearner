[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# lazylanguagelearner

Tùy chọn ngôn ngữ: **Tiếng Anh** | Các bản dịch README khác được lên kế hoạch trong [`i18n/`](i18n/)

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](../LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

Học ngôn ngữ theo cách lười.

## 🌍 Tổng quan

LazyLanguageLearner là một quy trình học ngôn ngữ dựa trên Python, kết hợp:

- Thu thập PDF cho các tài liệu nội dung khóa học Rosetta Stone.
- Phân tích PDF và trích xuất câu thành bộ dữ liệu CSV.
- Chuyển đổi câu đa ngôn ngữ bằng OpenAI, kèm cặp ngữ âm và bộ nhớ đệm cục bộ trên đĩa.
- Ứng dụng web Tornado gọn nhẹ để hiển thị văn bản đa ngôn ngữ với chú âm kiểu ruby.

Kho lưu trữ hiện tại vận hành theo kiểu script (chưa được đóng gói thành module pip), với các tệp dữ liệu và notebook được đưa trực tiếp vào repo.

## ✨ Tính năng

- Tải PDF khóa học ngôn ngữ từ các liên kết được nhúng trong `rs_html.py` (`download_course_text.py`).
- Trích xuất dữ liệu phần/câu từ PDF thành CSV có cấu trúc (`pdf_to_csv.py`, `language_extraction.py`).
- Lưu đệm dữ liệu prompt/response của OpenAI vào `cache/*.json` để giảm gọi API lặp lại (`openai_request.py`).
- Phân tích phản hồi AI thành JSON với logic thử lại và lỗi phân tích JSON tùy chỉnh.
- Phục vụ các khối câu đa ngôn ngữ từ `translations.json` qua Tornado (`app.py` + `templates/index.html`).
- Bao gồm chuẩn hóa ngữ âm tiếng Nhật (`katakana` sang `hiragana`) trước khi hiển thị.

## 🗂️ Cấu trúc dự án

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

## ✅ Điều kiện tiên quyết

Giả định (vì hiện chưa có lockfile hoặc dependency manifest được commit):

- Python 3.10+ (nhiều khả năng chạy được trên các phiên bản lân cận; ma trận đã kiểm thử chính xác chưa được công bố).
- `pip` và `venv`.
- OpenAI API key cho các script dùng model.

Các dependency Python được suy ra từ import:

| Package | Được dùng bởi |
|---|---|
| `tornado` | Web server trong `app.py` |
| `openai` | API calls trong `openai_request.py` |
| `PyPDF2` | PDF parsing trong các script trích xuất |
| `requests` | PDF downloads trong `download_course_text.py` |
| `beautifulsoup4` | HTML parsing trong downloader |

## 🛠️ Cài đặt

```bash
# from repository root
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

## 🚀 Cách dùng

### 1) Tải các PDF nguồn

```bash
python download_course_text.py
```

Lệnh này tạo `downloaded_pdfs/` và lưu các PDF ngôn ngữ/unit vào đó.

### 2) Trích xuất nội dung PDF tiếng Nhật sang CSV

```bash
python pdf_to_csv.py
```

Đầu ra mặc định theo script hiện tại: `japanese_language_data.csv`.

### 3) (Tùy chọn) Cắt văn bản section/page/sentence theo cách tương tác

```bash
python language_extraction.py
```

Script chứa các biến ví dụ có thể chỉnh sửa (`level`, `section`, `sentence_num`) và in ra văn bản đã trích xuất.

### 4) Tạo JSON đa ngôn ngữ bằng luồng OpenAI

```bash
python multilingual_sentence.py
```

Ghi chú hành vi hiện tại:

- Chỉ xử lý dòng CSV đầu tiên (`break` trong vòng lặp).
- Script hiện tham chiếu đến một biến chưa được định nghĩa (`japanese_text`) khi tạo prompt, nên cần sửa nhỏ trước khi dùng ổn định.

### 5) Chạy ứng dụng web

```bash
python app.py
```

- Tornado lắng nghe trên cổng `7788`.
- Mở trên trình duyệt: `http://localhost:7788/`.
- Lưu ý: log khởi động hiện in `http://localhost:8888` dù cổng bind là `7788`.

## ⚙️ Cấu hình

Biến môi trường:

| Variable | Required | Mục đích | Mặc định hiện tại |
|---|---|---|---|
| `OPENAI_API_KEY` | Yes | Bắt buộc cho khởi tạo client `OpenAI()` | N/A |
| `OPENAI_MODEL` | No | Tùy chọn ghi đè chat model | `gpt-4-0125-preview` |

Tệp/thư mục runtime:

- `downloaded_pdfs/`: được tạo bởi downloader, dùng bởi các script trích xuất.
- `cache/`: bộ nhớ đệm request/response cho các cuộc gọi OpenAI.
- `translations.json`: nguồn dữ liệu để Tornado UI hiển thị.

## 🧾 Ví dụ định dạng dữ liệu

### CSV (`japanese_language_data.csv`)

Header được dùng bởi `pdf_to_csv.py`:

```csv
Level,Unit,Section,Sentence No.,Content
```

### JSON (`translations.json`)

Web UI mong đợi các khóa ngôn ngữ có mục `pairs` chứa `part` và `phonetic`:

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

## 🧪 Ghi chú phát triển

- Repo hiện chưa có `requirements.txt`, `pyproject.toml`, hoặc workflow CI.
- Các script được thiết kế để chạy trực tiếp từ repository root.
- Các notebook hiện có (`*.ipynb`) có vẻ thiên về khám phá/prototype.
- Các artifact CSV lớn được version trực tiếp trong Git.
- `i18n/` đã tồn tại và sẵn sàng cho các biến thể README đã dịch.

## 🩺 Khắc phục sự cố

- `ModuleNotFoundError`: cài các dependency suy ra trong virtual environment đang kích hoạt.
- Lỗi auth `OPENAI`: đảm bảo `OPENAI_API_KEY` đã được export trong shell.
- `FileNotFoundError: downloaded_pdfs`: chạy `python download_course_text.py` trước.
- Lỗi `multilingual_sentence.py` tại `japanese_text`: thay `japanese_text` bằng `content` khi tạo prompt.
- Nhầm cổng app: dùng `http://localhost:7788/` trừ khi `app.listen(...)` được thay đổi.

## 🛣️ Lộ trình

Các bước tiếp theo tiềm năng cho dự án:

- Thêm dependency manifest (`requirements.txt` hoặc `pyproject.toml`).
- Sửa biến prompt trong `multilingual_sentence.py` và bỏ `break` một dòng để xử lý theo lô.
- Đồng bộ URL in khi khởi động Tornado với cổng bind.
- Thêm test cho hành vi regex trích xuất PDF và logic phân tích/thử lại JSON.
- Thêm tham số CLI cho ngôn ngữ/cấp độ/đường dẫn để giảm sửa trực tiếp trong file.
- Điền các tệp README đã dịch vào `i18n/`.

## 🤝 Đóng góp

Rất hoan nghênh đóng góp.

1. Fork repository.
2. Tạo feature branch.
3. Thực hiện thay đổi tập trung với commit message rõ ràng.
4. Mở pull request mô tả những gì đã thay đổi và lý do.

Nếu bạn sửa logic trích xuất, hãy kèm các đoạn ví dụ input/output để việc review dễ hơn.

## 🙏 Lời cảm ơn

- Các liên kết nội dung khóa học Rosetta Stone được nhúng trong `rs_html.py` và dùng làm tham chiếu nguồn để tải PDF.
- OpenAI API được dùng cho việc tạo nội dung đa ngôn ngữ và cấu trúc ngữ âm.

## 📄 Giấy phép

Dự án này được cấp phép theo Apache License 2.0. Xem [LICENSE](../LICENSE).
