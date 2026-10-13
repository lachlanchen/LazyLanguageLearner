[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://lazying.art)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| Thuộc tính | Giá trị |
|---|---|
| Kiểu | Quy trình học ngôn ngữ đa ngôn ngữ do script điều phối |
| Runtime | Python CLI + ứng dụng web Tornado |
| Nguồn chính | PDF khóa học Rosetta Stone |
| Lưu trữ | CSV cục bộ + tệp cache JSON |
| Cổng mặc định | `7788` |

LazyLanguageLearner là một quy trình làm việc Python theo kiểu script-driven để chuyển đổi PDF khóa học ngôn ngữ thành dữ liệu học đa ngôn ngữ có thể dùng lại, sau đó hiển thị trong giao diện web tối giản.

## 🌍 Tổng quan

Kho này kết hợp ba giai đoạn trích xuất nội dung, biến đổi và phục vụ:

| Bước | Mục đích |
|---|---|
| 1 | Tải các PDF nội dung khóa học Rosetta Stone từ các liên kết nhúng trong `rs_html.py`. |
| 2 | Phân tích PDF thành các hàng CSV ở cấp độ câu. |
| 3 | Tạo các biến thể đa ngôn ngữ/chữ phiên âm thông qua OpenAI và lưu cache trên đĩa. |
| 4 | Render các câu đã cấu trúc trong giao diện web Tornado với chú thích phiên âm. |

Dự án này được thiết kế nhẹ nhàng và tập trung ở root: các script được chạy trực tiếp từ thư mục gốc của repo thay vì đóng gói thành package.

## ✨ Tính năng

- **Luồng tải xuống tự động** từ các liên kết nhúng trong `rs_html.py` bằng `download_course_text.py`.
- **Pipeline trích xuất bằng regex + PDF** cho phần/đoạn/văn bản câu trong `pdf_to_csv.py`.
- **Công cụ trích xuất chọn lọc** cho mức độ, phần, trang và kiểm tra câu trong `language_extraction.py`.
- **Lớp gọi OpenAI** (`openai_request.py`) với cơ chế tra cache, xử lý prompt và thử lại cơ bản khi parse JSON.
- **Pipeline render liên ngôn ngữ** do `app.py` và `templates/index.html` phục vụ.
- **Chuẩn hóa phiên âm tiếng Nhật** bằng cách chuyển dữ liệu katakana sang hiragana trước khi render.
- **Cache trên đĩa** tại `cache/` cho các yêu cầu/đáp ứng dịch được tạo ra.

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
│   └── README.*.md
└── *.ipynb
```

## ✅ Yêu cầu trước khi chạy

- Python `3.10+`
- `pip` cùng môi trường ảo hoạt động (`venv` được khuyên dùng)
- Khóa API OpenAI (`OPENAI_API_KEY`) khi dùng các luồng tạo nội dung AI
- Kết nối internet hoạt động để tải PDF và gọi API OpenAI

Vì repo này chưa có lockfile, các phụ thuộc được suy ra từ import và nội dung trước đó:

| Gói | Được dùng bởi |
|---|---|
| `tornado` | `app.py` |
| `openai` | `openai_request.py`, `multilingual_sentence.py` |
| `PyPDF2` | `pdf_to_csv.py`, `language_extraction.py` |
| `requests` | `download_course_text.py` |
| `beautifulsoup4` | `download_course_text.py` |

## 🛠️ Cài đặt

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

| Gợi ý thiết lập | Lệnh |
|---|---|
| Kích hoạt venv | `source .venv/bin/activate` |
| Tái tạo môi trường | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| Chạy kiểm tra | `python -m pip check` |

## 🚀 Cách sử dụng

Chạy các script theo đúng thứ tự sau cho pipeline chuẩn:

### 1) Tải nguồn PDF

```bash
python download_course_text.py
```

Tải PDF vào `downloaded_pdfs/`.

### 2) Trích xuất nội dung PDF ra CSV

```bash
python pdf_to_csv.py
```

Mặc định tạo file `japanese_language_data.csv`.

### 3) Kiểm tra một lát cắt PDF cụ thể (tùy chọn)

```bash
python language_extraction.py
```

Hữu ích khi xác minh `level`, `section`, `page`, và `sentence_num` cụ thể trước khi tạo dataset quy mô lớn.

### 4) Tạo payload câu đa ngôn ngữ (tùy chọn)

```bash
python multilingual_sentence.py
```

Ghi chú hành vi hiện tại để tăng độ tin cậy:

- Phiên bản hiện tại chỉ xử lý đúng một hàng đầu tiên do `break` sớm.
- Việc tạo prompt đang tham chiếu đến `japanese_text`, giá trị này hiện chưa nhất quán với biến hàng CSV đã trích xuất và có thể gây lỗi.

### 5) Khởi chạy web app

```bash
python app.py
```

- Cổng mặc định của Tornado: `7788`
- URL: `http://localhost:7788/`
- Sai lệch đã biết cần kiểm tra log: thông báo khởi động hiện tại đang tham chiếu `http://localhost:8888`.

## ⚙️ Cấu hình

Các biến môi trường được script chạy ở runtime kỳ vọng:

| Biến | Bắt buộc | Mục đích | Mặc định |
|---|---|---|---|
| `OPENAI_API_KEY` | Có (chỉ luồng AI) | Xác thực OpenAI | N/A |
| `OPENAI_MODEL` | Không | Ghi đè model trong request | `gpt-4-0125-preview` |

File/thư mục runtime:

- `downloaded_pdfs/` — được `download_course_text.py` điền dữ liệu.
- `cache/` — lưu payload prompt/response OpenAI đã cache.
- `translations.json` — được Tornado UI sử dụng.
- `templates/index.html` — template render trên trình duyệt.

Giả định:

- Thư mục gốc repository là nơi làm việc dự kiến cho tất cả script.
- Cache dịch có thể được tái tạo an toàn khi đã cũ hoặc thiếu.

## 🧾 Ví dụ

### Định dạng CSV (`japanese_language_data.csv`)

```csv
Level,Unit,Section,Sentence No.,Content
```

### Dạng payload dịch OpenAI (`translations.json`)

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

### Kiểm tra nhanh tối thiểu

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 Ghi chú phát triển

- Dự án không được đóng gói (`requirements.txt`, `pyproject.toml` và CI không có).
- Script-first: script được viết để chỉnh sửa và chạy lại trong quá trình lặp.
- Các file notebook mang tính thăm dò và nên dùng như công cụ nghiên cứu, không phải pipeline production.
- `i18n/README.*.md` đã có cho tài liệu đa ngôn ngữ, với block điều hướng ngôn ngữ cấp cao trong file này hoạt động như điểm vào chung.

## 🩺 Khắc phục sự cố

- `ModuleNotFoundError`: cài đủ các package cần thiết trong môi trường ảo đang dùng.
- Lỗi xác thực `OPENAI` / phản hồi rỗng: kiểm tra biến `OPENAI_API_KEY` đã được export trong shell.
- `FileNotFoundError` cho `downloaded_pdfs`: chạy `python download_course_text.py` trước.
- Vấn đề chuyển đổi OpenAI: kiểm tra `cache/*.json` và xác minh định dạng payload đầu vào mà `multilingual_sentence.py` kỳ vọng.
- Nhầm lẫn URL ứng dụng: mở `http://localhost:7788/` sau khi khởi chạy.

## 🛣️ Lộ trình

- Thêm manifest phụ thuộc (`requirements.txt` hoặc `pyproject.toml`) để cài đặt tái lập.
- Xóa `break` xử lý một hàng trong `multilingual_sentence.py` và hỗ trợ sinh đa lô đầy đủ.
- Sửa lỗi dùng biến trong prompt của `multilingual_sentence.py` và thêm validation đầu ra.
- Sửa log URL khởi động của Tornado để khớp cổng `7788`.
- Thêm các flag CLI (ngôn ngữ, cấp độ, đường dẫn nguồn, đường dẫn output).
- Bổ sung test nhẹ cho extraction, retry/parse logic, và validate schema JSON.
- Mở rộng tài liệu dành cho người đóng góp trong các biến thể ngôn ngữ của `i18n`.

## 🤝 Đóng góp

Mọi đóng góp đều được chào đón.

1. Fork repository.
2. Tạo nhánh feature.
3. Thực hiện thay đổi có trọng tâm và giữ quy trình script dễ tái tạo.
4. Mở pull request với lý do rõ ràng và ghi chú hành vi trước/sau.

Nếu bạn cập nhật logic trích xuất, hãy bao gồm dữ liệu đầu vào/đầu ra mẫu trong mô tả PR.

## 🙏 Ghi nhận

- Các liên kết nội dung khóa học Rosetta Stone trong `rs_html.py` là nguồn tham chiếu corpus PDF có thể tải về.
- API OpenAI được dùng cho thử nghiệm tạo đa ngôn ngữ và gắn chú thích phiên âm.

## 📄 Giấy phép

Dự án này được cấp phép theo Apache License 2.0. Xem [LICENSE](LICENSE).


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
