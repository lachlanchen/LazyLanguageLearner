[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner — apps and four buddies](https://lazying.art/media/brand/lazyingart-intro.gif)](https://lazying.art)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| Атрибут | Значение |
|---|---|
| Тип | Скрипт-ориентированный многоязычный языковой конвейер |
| Среда выполнения | Python CLI + веб-приложение Tornado |
| Основной источник | PDF-курсы Rosetta Stone |
| Хранилище | Локальные кэши CSV + JSON |
| Порт по умолчанию | `7788` |

LazyLanguageLearner — это скриптовый Python-конвейер для превращения PDF-курсов по языкам в многоразовые многоязычные учебные данные с последующим отображением в минималистичном веб-интерфейсе.

## 🌍 Обзор

Репозиторий объединяет извлечение, преобразование и выдачу контента:

| Этап | Назначение |
|---|---|
| 1 | Скачивает PDF-файлы с содержимым курсов Rosetta Stone по ссылкам из `rs_html.py`. |
| 2 | Разбирает PDF в строки CSV на уровне предложений для дальнейшего преобразования. |
| 3 | Генерирует многоязычные/фонетические варианты через OpenAI с кэшированием на диске. |
| 4 | Отображает структурированные предложения в веб-интерфейсе Tornado с фонетическими аннотациями. |

Проект намеренно лёгкий и ориентирован на корень репозитория: скрипты рассчитаны на запуск прямо из корня, а не как установленный пакет.

## ✨ Возможности

- **Автоматизированный процесс скачивания** из встроенных ссылок в `rs_html.py` с помощью `download_course_text.py`.
- **Конвейер извлечения PDF** на основе regex в `pdf_to_csv.py` для выделения секций и предложений.
- **Утилиты выборочного извлечения** для уровня, раздела, страницы и отдельного предложения в `language_extraction.py`.
- **Слой работы с OpenAI** (`openai_request.py`) с поиском в кеше, управлением prompt и базовыми повторными попытками JSON-вывода.
- **Кросс-языковой конвейер рендеринга**, реализованный через `app.py` и `templates/index.html`.
- **Нормализация японской фонетики**: преобразование катаканы в хирагану перед рендером.
- **Кэширование на диске** в `cache/` для сгенерированных запросов и ответов переводчика.

## 🗂️ Структура проекта

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

## ✅ Предварительные требования

- Python `3.10+`
- `pip` с активным виртуальным окружением (`venv` рекомендуется)
- Ключ OpenAI API (`OPENAI_API_KEY`) для AI-поддерживаемой генерации
- Доступ в интернет для скачивания PDF и запросов к OpenAI

Поскольку в репозитории нет lockfile, зависимости определяются по импортам и текущему содержимому:

| Пакет | Используется в |
|---|---|
| `tornado` | `app.py` |
| `openai` | `openai_request.py`, `multilingual_sentence.py` |
| `PyPDF2` | `pdf_to_csv.py`, `language_extraction.py` |
| `requests` | `download_course_text.py` |
| `beautifulsoup4` | `download_course_text.py` |

## 🛠️ Установка

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

| Подсказка установки | Команда |
|---|---|
| Активировать venv | `source .venv/bin/activate` |
| Воссоздать окружение | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| Проверить окружение | `python -m pip check` |

## 🚀 Использование

Запускайте скрипты в этом порядке для стандартного конвейера:

### 1) Скачивание исходных PDF

```bash
python download_course_text.py
```

Скачивает PDF-файлы в `downloaded_pdfs/`.

### 2) Извлечение PDF в CSV

```bash
python pdf_to_csv.py
```

По умолчанию создаёт `japanese_language_data.csv`.

### 3) Просмотр конкретного фрагмента PDF (необязательно)

```bash
python language_extraction.py
```

Полезно для проверки конкретных путей `level`, `section`, `page`, `sentence_num` перед генерацией более широких наборов.

### 4) Построение многоязычных пакетов предложений (необязательно)

```bash
python multilingual_sentence.py
```

Текущие замечания по надёжности:

- Текущая версия обрабатывает только первую строку из‑за раннего `break`.
- Формирование prompt ссылается на `japanese_text`, что сейчас может конфликтовать с переменной строки после извлечения и привести к сбою.

### 5) Запуск веб-приложения

```bash
python app.py
```

- Порт Tornado по умолчанию: `7788`
- URL: `http://localhost:7788/`
- Известное расхождение, которое стоит проверить в логах: при запуске печатается `http://localhost:8888`.

## ⚙️ Конфигурация

Переменные окружения, которые ожидаются скриптами исполнения:

| Переменная | Обязательна | Назначение | Значение по умолчанию |
|---|---|---|---|
| `OPENAI_API_KEY` | Да (только AI-флоу) | Аутентификация OpenAI | N/A |
| `OPENAI_MODEL` | Нет | Переопределение модели в запросах | `gpt-4-0125-preview` |

Файлы и директории во время выполнения:

- `downloaded_pdfs/` — заполняется `download_course_text.py`.
- `cache/` — хранит кэшированные payloadы prompt/response OpenAI.
- `translations.json` — используется веб-интерфейсом Tornado.
- `templates/index.html` — шаблон рендеринга в браузере.

Предположения:

- Корень репозитория является рабочей директорией для всех скриптов.
- Кэш переводов можно безопасно пересоздать, если он устарел или отсутствует.

## 🧾 Примеры

### Формат CSV (`japanese_language_data.csv`)

```csv
Level,Unit,Section,Sentence No.,Content
```

### Схема payload OpenAI `translations.json`

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

### Минимальная быстрая проверка

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 Заметки по разработке

- Проект не является пакетированным (`requirements.txt`, `pyproject.toml` и CI не добавлены).
- Скрипты ориентированы на прямую доработку и многократный запуск в процессе итераций.
- Файлы блокнотов выглядят исследовательскими и лучше использовать как инструмент верификации, а не как боевой конвейер.
- `i18n/README.*.md` уже существует для многоязычной документации, при этом блок переключения языков в этом файле служит общей точкой входа.

## 🩺 Устранение неполадок

- `ModuleNotFoundError`: установите все необходимые пакеты в активное виртуальное окружение.
- Ошибка авторизации `OPENAI` / пустые ответы: проверьте, что `OPENAI_API_KEY` экспортирован в вашей оболочке.
- `FileNotFoundError` для `downloaded_pdfs`: сначала запустите `python download_course_text.py`.
- Проблемы с конвертацией OpenAI: проверьте `cache/*.json` и убедитесь, что формат входного payload соответствует ожиданиям `multilingual_sentence.py`.
- Неясность адреса приложения: после запуска откройте `http://localhost:7788/`.

## 🛣️ Дорожная карта

- Добавить манифест зависимостей (`requirements.txt` или `pyproject.toml`) для воспроизводимой установки.
- Удалить прерывание на одной строке (`break`) из `multilingual_sentence.py` и добавить поддержку пакетной многоязыковой генерации.
- Исправить использование переменной в prompt в `multilingual_sentence.py` и добавить валидацию вывода.
- Исправить лог URL запуска Tornado, чтобы в нём был порт `7788`.
- Добавить CLI-параметры (язык, уровень, исходные пути, путь вывода).
- Ввести лёгкие тесты для извлечения, retry/parse-логики и валидации JSON-схем.
- Расширить документацию для контрибьютеров в языковых вариантах `i18n`.

## 🤝 Участие в разработке

Добро пожаловать к участию.

1. Форкните репозиторий.
2. Создайте feature branch.
3. Вносите целевые изменения и поддерживайте воспроизводимость скриптовых workflow.
4. Откройте pull request с понятным обоснованием и описанием поведения до/после.

Если вы меняете логику извлечения, приложите примеры входных и выходных данных в описании PR.

## 🙏 Благодарности

- Ссылки на курсы Rosetta Stone в `rs_html.py` являются источником ссылок на PDF-корпус для загрузки.
- API OpenAI используется для экспериментов с многоязычной генерацией и фонетической аннотацией.

## 📄 Лицензия

Этот проект распространяется по лицензии Apache License 2.0. Смотрите файл [LICENSE](LICENSE).


## ❤️ Support

<!-- support:start -->
[![GitHub Sponsors](https://img.shields.io/badge/GitHub-Sponsors-EA4AAA?style=for-the-badge&logo=githubsponsors&logoColor=white)](https://github.com/sponsors/lachlanchen)
[![LazyingArt Support](https://img.shields.io/badge/LazyingArt-Support-0EA5E9?style=for-the-badge)](https://chat.lazying.art/donate)
[![PayPal Support](https://img.shields.io/badge/PayPal-Support-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen)
[![Stripe Support](https://img.shields.io/badge/Stripe-Support-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400)
<!-- support:end -->
