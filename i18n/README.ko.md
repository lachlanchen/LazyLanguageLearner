[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://lazying.art)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| 항목 | 값 |
|---|---|
| 유형 | 스크립트 기반 다국어 학습 파이프라인 |
| 런타임 | Python CLI + Tornado 웹 앱 |
| 주요 소스 | Rosetta Stone 코스 PDF |
| 저장소 | 로컬 CSV + JSON 캐시 파일 |
| 기본 포트 | `7788` |

LazyLanguageLearner는 언어 코스 PDF를 재사용 가능한 다국어 학습 데이터로 변환하고, 최소한의 웹 UI에서 렌더링하는 스크립트 기반 Python 워크플로입니다.

## 🌍 개요

이 저장소는 콘텐츠 추출, 변환, 제공을 결합합니다.

| 단계 | 목적 |
|---|---|
| 1 | `rs_html.py`에 포함된 링크에서 Rosetta Stone 코스 콘텐츠 PDF를 다운로드합니다. |
| 2 | PDF를 문장 단위 CSV 행으로 파싱합니다. |
| 3 | OpenAI를 사용해 다국어/음성 표기 변형을 생성하고, 디스크 캐시로 저장합니다. |
| 4 | 정제된 문장을 Tornado 웹 UI에서 음성 주석과 함께 렌더링합니다. |

이 프로젝트는 의도적으로 가볍고 루트 중심적입니다. 스크립트는 설치된 패키지로 실행되는 것이 아니라 저장소 루트에서 직접 실행되도록 설계되었습니다.

## ✨ 기능

- `download_course_text.py`를 통해 `rs_html.py`에 포함된 링크에서 PDF를 자동으로 다운로드합니다.
- `pdf_to_csv.py`에서 구간/문장 추출을 위한 정규식 + PDF 추출 파이프라인을 제공합니다.
- `language_extraction.py`에서 단계별(level), 섹션, 페이지, 문장 조회를 위한 선택적 추출 유틸리티를 제공합니다.
- 캐시 조회, 프롬프트 처리, 기본 JSON 추출 재시도 로직이 포함된 `openai_request.py`의 OpenAI 요청 계층을 제공합니다.
- `app.py`와 `templates/index.html`을 통해 다국어 렌더링 파이프라인을 제공합니다.
- 렌더링 전 가타카나 데이터를 히라가나로 변환하는 일본어 음성 정규화를 적용합니다.
- 생성된 번역 요청/응답은 `cache/` 하위 디렉터리의 디스크 캐시로 저장되어 재사용됩니다.

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
│   └── README.*.md
└── *.ipynb
```

## ✅ 필수 조건

- Python `3.10+`
- `pip` 및 가상 환경(`venv`) 사용 권장
- AI 기반 생성 시 OpenAI API 키(`OPENAI_API_KEY`)
- PDF 다운로드와 OpenAI 요청을 위한 인터넷 연결

이 저장소에는 lockfile이 없으므로 의존성은 import와 기존 내용으로 추론됩니다.

| 패키지 | 사용 위치 |
|---|---|
| `tornado` | `app.py` |
| `openai` | `openai_request.py`, `multilingual_sentence.py` |
| `PyPDF2` | `pdf_to_csv.py`, `language_extraction.py` |
| `requests` | `download_course_text.py` |
| `beautifulsoup4` | `download_course_text.py` |

## 🛠️ 설치

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

| 설정 팁 | 명령 |
|---|---|
| 가상 환경 활성화 | `source .venv/bin/activate` |
| 동일 환경 재현 | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| 점검 실행 | `python -m pip check` |

## 🚀 사용법

표준 파이프라인은 아래 순서로 실행합니다.

### 1) 소스 PDF 다운로드

```bash
python download_course_text.py
```

`downloaded_pdfs/`에 PDF가 저장됩니다.

### 2) PDF 콘텐츠를 CSV로 추출

```bash
python pdf_to_csv.py
```

기본값으로 `japanese_language_data.csv`를 출력합니다.

### 3) 특정 PDF 조각 검사(선택)

```bash
python language_extraction.py
```

더 넓은 데이터셋을 생성하기 전에 특정 `level`, `section`, `page`, `sentence_num` 경로를 검증할 때 유용합니다.

### 4) 다국어 문장 페이로드 생성(선택)

```bash
python multilingual_sentence.py
```

신뢰성 관련 현재 동작:

- 현재 버전은 조기 `break`로 인해 첫 번째 행만 처리합니다.
- 프롬프트 생성이 현재 추출된 CSV 행 변수와 일치하지 않을 가능성이 있는 `japanese_text`를 참조해 실패할 수 있습니다.

### 5) 웹 앱 실행

```bash
python app.py
```

- 기본 Tornado 포트: `7788`
- URL: `http://localhost:7788/`
- 로그에서 확인해야 할 알려진 불일치: 시작 메시지가 현재 `http://localhost:8888`을 참조합니다.

## ⚙️ 설정

런타임 스크립트에서 기대하는 환경 변수:

| 변수 | 필수 | 목적 | 기본값 |
|---|---|---|---|
| `OPENAI_API_KEY` | 예 (AI 흐름만 해당) | OpenAI 인증 | N/A |
| `OPENAI_MODEL` | 아니오 | 요청의 모델 오버라이드 | `gpt-4-0125-preview` |

런타임 파일/디렉터리:

- `downloaded_pdfs/` — `download_course_text.py`에서 채웁니다.
- `cache/` — OpenAI 프롬프트/응답 페이로드를 캐시합니다.
- `translations.json` — Tornado UI에서 사용됩니다.
- `templates/index.html` — 브라우저 렌더링 템플릿.

가정:

- 모든 스크립트의 작업 디렉터리는 저장소 루트입니다.
- 번역 캐시는 오래되었거나 누락된 경우 안전하게 다시 생성할 수 있습니다.

## 🧾 예시

### CSV 형식 (`japanese_language_data.csv`)

```csv
Level,Unit,Section,Sentence No.,Content
```

### OpenAI 번역 페이로드 형식 (`translations.json`)

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

### 최소 동작 확인

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 개발 노트

- 프로젝트는 패키지화되지 않았습니다(`requirements.txt`, `pyproject.toml`, CI가 없음).
- 스크립트 우선 설계로, 반복 시 편집 후 재실행하는 방식이 적합합니다.
- 노트북 파일은 탐색용으로 보이며, 운영 파이프라인이 아닌 연구용 보조 자료로 다루는 것이 좋습니다.
- `i18n/README.*.md`는 이미 존재하며 이 파일의 상단 언어 전환 링크가 공통 진입점 역할을 합니다.

## 🩺 문제 해결

- `ModuleNotFoundError`: 가상 환경에 필요한 모든 패키지를 설치하세요.
- `OPENAI` 인증 오류/빈 응답: 셸에서 `OPENAI_API_KEY`가 export되어 있는지 확인하세요.
- `downloaded_pdfs`에서 `FileNotFoundError`: 먼저 `python download_course_text.py`를 실행하세요.
- OpenAI 변환 문제: `cache/*.json`을 확인하고 `multilingual_sentence.py`에서 기대하는 입력 페이로드 형식을 검증하세요.
- 앱 URL 혼동: 실행 후 `http://localhost:7788/`로 접속하세요.

## 🛣️ 로드맵

- 재현 가능한 설치를 위해 종속성 매니페스트(`requirements.txt` 또는 `pyproject.toml`)를 추가합니다.
- `multilingual_sentence.py`의 단일 행 `break`를 제거하고 전체 배치 다국어 생성을 지원합니다.
- `multilingual_sentence.py`의 프롬프트 변수 사용을 수정하고 출력 검증을 추가합니다.
- Tornado 시작 URL 로그를 실제 포트 `7788`로 맞춥니다.
- CLI 플래그(언어, 레벨, 소스 경로, 출력 경로)를 추가합니다.
- 추출, 재시도/파싱 로직, JSON 스키마 검증을 위한 경량 테스트를 도입합니다.
- `i18n`의 언어별 문서 범위를 확장합니다.

## 🤝 기여

기여를 환영합니다.

1. 저장소를 포크합니다.
2. 기능 브랜치를 생성합니다.
3. 변경 범위를 좁게 하고 스크립트 워크플로의 재현성을 유지합니다.
4. 변경 이유가 명확한 풀 리퀘스트를 제출하고 변경 전/후 동작 차이를 정리합니다.

추출 로직을 업데이트할 때는 PR 설명에 샘플 입력과 출력을 포함하세요.

## 🙏 감사의 말

- `rs_html.py`의 Rosetta Stone 코스 콘텐츠 링크는 다운로드 가능한 PDF 코퍼스의 참조 출처입니다.
- OpenAI API는 다국어 생성과 음성 주석 실험에 사용되었습니다.

## 📄 라이선스

이 프로젝트는 Apache License 2.0 아래에서 라이선스가 부여됩니다. 자세한 내용은 [LICENSE](LICENSE)를 참조하세요.


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
