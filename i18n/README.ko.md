[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# lazylanguagelearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

게으른 방식으로 언어를 학습하세요.

## 🌍 개요

LazyLanguageLearner는 다음을 결합한 Python 기반 언어 학습 워크플로우입니다.

- Rosetta Stone 코스 콘텐츠 문서용 PDF 수집
- PDF 파싱 및 문장 추출을 통한 CSV 데이터셋 생성
- OpenAI 기반 다국어 문장 변환, 음성 쌍(phonetic pairs), 로컬 디스크 캐시
- 루비(ruby) 발음 주석이 포함된 다국어 텍스트를 렌더링하는 경량 Tornado 웹 앱

현재 저장소는 스크립트 중심 구조이며(아직 pip 모듈로 패키징되지 않음), 데이터 파일과 노트북이 저장소에 직접 포함되어 있습니다.

## ✨ 기능

- `rs_html.py`에 포함된 링크에서 언어 코스 PDF를 다운로드 (`download_course_text.py`)
- PDF에서 섹션/문장 데이터를 구조화된 CSV로 추출 (`pdf_to_csv.py`, `language_extraction.py`)
- 반복 API 사용을 줄이기 위해 OpenAI 프롬프트/응답 데이터를 `cache/*.json`에 캐시 (`openai_request.py`)
- 재시도 로직 및 커스텀 JSON 파싱 에러를 포함해 AI 응답을 JSON으로 파싱
- Tornado를 통해 `translations.json`의 다국어 문장 블록 제공 (`app.py` + `templates/index.html`)
- 렌더링 전 일본어 발음 정규화(`katakana` -> `hiragana`) 포함

## 🗂️ 프로젝트 구조

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

## ✅ 사전 요구 사항

가정 사항(현재 lockfile 또는 의존성 매니페스트가 커밋되어 있지 않음):

- Python 3.10+ (근접 버전에서도 동작할 가능성 높음, 정확한 테스트 매트릭스는 명시되지 않음)
- `pip` 및 `venv`
- 모델 기반 스크립트를 위한 OpenAI API 키

import 기준으로 추론한 Python 의존성:

| Package | Used by |
|---|---|
| `tornado` | `app.py`의 웹 서버 |
| `openai` | `openai_request.py`의 API 호출 |
| `PyPDF2` | 추출 스크립트의 PDF 파싱 |
| `requests` | `download_course_text.py`의 PDF 다운로드 |
| `beautifulsoup4` | 다운로더의 HTML 파싱 |

## 🛠️ 설치

```bash
# from repository root
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

## 🚀 사용 방법

### 1) 원본 PDF 다운로드

```bash
python download_course_text.py
```

이 명령은 `downloaded_pdfs/`를 생성하고 언어/유닛 PDF를 해당 경로에 저장합니다.

### 2) 일본어 PDF 콘텐츠를 CSV로 추출

```bash
python pdf_to_csv.py
```

현재 스크립트 기준 기본 출력 파일: `japanese_language_data.csv`.

### 3) (선택) 섹션/페이지/문장 텍스트를 대화형으로 추출

```bash
python language_extraction.py
```

이 스크립트에는 수정 가능한 예시 변수(`level`, `section`, `sentence_num`)가 있으며, 추출된 텍스트를 출력합니다.

### 4) OpenAI 흐름으로 다국어 JSON 생성

```bash
python multilingual_sentence.py
```

현재 동작 참고 사항:

- 첫 번째 CSV 행만 처리됩니다(루프 내 `break`).
- 스크립트가 프롬프트 생성 시 정의되지 않은 변수(`japanese_text`)를 참조하므로, 안정적으로 사용하려면 작은 수정이 필요합니다.

### 5) 웹 앱 실행

```bash
python app.py
```

- Tornado는 `7788` 포트에서 수신합니다.
- 브라우저에서 열기: `http://localhost:7788/`.
- 참고: 현재 시작 로그는 `7788`에 바인딩되어 있음에도 `http://localhost:8888`을 출력합니다.

## ⚙️ 설정

환경 변수:

| Variable | Required | Purpose | Current default |
|---|---|---|---|
| `OPENAI_API_KEY` | Yes | `OpenAI()` 클라이언트 초기화에 필요 | N/A |
| `OPENAI_MODEL` | No | 채팅 모델 선택값 오버라이드(선택) | `gpt-4-0125-preview` |

런타임 파일/디렉터리:

- `downloaded_pdfs/`: 다운로더가 생성하며 추출 스크립트에서 사용
- `cache/`: OpenAI 호출용 요청/응답 캐시
- `translations.json`: Tornado UI 렌더링 데이터 소스

## 🧾 데이터 형식 예시

### CSV (`japanese_language_data.csv`)

`pdf_to_csv.py`에서 사용하는 헤더:

```csv
Level,Unit,Section,Sentence No.,Content
```

### JSON (`translations.json`)

웹 UI는 `part`와 `phonetic`을 포함한 `pairs` 항목이 있는 언어 키를 기대합니다.

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

## 🧪 개발 메모

- 이 저장소에는 현재 `requirements.txt`, `pyproject.toml`, CI 워크플로우가 없습니다.
- 스크립트는 저장소 루트에서 직접 실행하도록 설계되었습니다.
- 기존 노트북(`*.ipynb`)은 탐색/프로토타이핑 성격으로 보입니다.
- 큰 CSV 산출물이 Git에 직접 버전 관리됩니다.
- `i18n/`은 번역 README 변형을 위한 준비가 되어 있습니다.

## 🩺 문제 해결

- `ModuleNotFoundError`: 활성 가상환경에 추론된 의존성을 설치하세요.
- `OPENAI` 인증 오류: 셸에서 `OPENAI_API_KEY`가 export 되었는지 확인하세요.
- `FileNotFoundError: downloaded_pdfs`: 먼저 `python download_course_text.py`를 실행하세요.
- `multilingual_sentence.py`의 `japanese_text` 관련 실패: 프롬프트 구성에서 `japanese_text`를 `content`로 교체하세요.
- 앱 포트 혼동: `app.listen(...)`을 바꾸지 않았다면 `http://localhost:7788/`을 사용하세요.

## 🛣️ 로드맵

프로젝트의 잠재적인 다음 단계:

- 의존성 매니페스트(`requirements.txt` 또는 `pyproject.toml`) 추가
- `multilingual_sentence.py`의 프롬프트 변수 수정 및 일괄 처리를 위해 1행 `break` 제거
- Tornado 시작 출력 URL과 실제 바인딩 포트 정렬
- PDF 추출 정규식 동작과 JSON 파싱/재시도 로직 테스트 추가
- 파일 내 직접 수정 부담을 줄이기 위한 언어/레벨/경로 CLI 인자 추가
- `i18n/`에 번역 README 파일 채우기

## 🤝 기여

기여를 환영합니다.

1. 저장소를 포크합니다.
2. 기능 브랜치를 생성합니다.
3. 명확한 커밋 메시지와 함께 변경 범위를 집중해서 작업합니다.
4. 무엇이 왜 바뀌었는지 설명하는 Pull Request를 엽니다.

추출 로직을 수정한 경우, 리뷰를 쉽게 할 수 있도록 샘플 입력/출력 스니펫을 포함해 주세요.

## 🙏 감사의 말

- Rosetta Stone 코스 콘텐츠 링크는 `rs_html.py`에 포함되어 있으며 PDF 다운로드의 소스 참조로 사용됩니다.
- OpenAI API는 다국어 생성 및 음성(phonetic) 구조화에 사용됩니다.

## 📄 라이선스

이 프로젝트는 Apache License 2.0에 따라 라이선스가 부여됩니다. 자세한 내용은 [LICENSE](LICENSE)를 참조하세요.
