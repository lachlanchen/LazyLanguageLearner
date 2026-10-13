[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://lazying.art)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| السمة | القيمة |
|---|---|
| النوع | خط أنابيب تعلم اللغات متعدد اللغات قائم على السكربت |
| زمن التشغيل | واجهة Python CLI + تطبيق ويب Tornado |
| المصدر الرئيسي | ملفات PDF لدورات Rosetta Stone |
| التخزين | ملفات CSV وJSON المؤقتة محليًا |
| المنفذ الافتراضي | `7788` |

LazyLanguageLearner هو سير عمل مكتوب بـ Python يحوّل ملفات PDF الخاصة بدورات اللغة إلى بيانات تعليم لغات متعددة الاستخدامات، ويعرضها في واجهة ويب بسيطة.

## 🌍 نظرة عامة

يجمع هذا المستودع بين استخراج المحتوى والتحويل والخدمات:

| الخطوة | الهدف |
|---|---|
| 1 | تنزيل ملفات PDF الخاصة بمحتوى دورات Rosetta Stone من الروابط المضمنة في `rs_html.py`. |
| 2 | تحليل ملفات PDF إلى صفوف CSV على مستوى الجملة للتحويل. |
| 3 | توليد نسخ متعددة اللغات/فونيتيكية عبر OpenAI مع استخدام ذاكرة تخزين مؤقت على القرص. |
| 4 | عرض الجمل المنظمة في واجهة Tornado مع تعليقات فونيتيكية. |

المشروع خفيف الوزن بشكل مقصود ومتمركز على الجذر: صُممت السكربتات لتُشغَّل مباشرة من جذر المستودع بدلًا من استخدام حزمة مثبّتة.

## ✨ الميزات

- **سير عمل تنزيل آلي** من الروابط المضمنة في `rs_html.py` باستخدام `download_course_text.py`.
- **خط أنابيب استخراج بـ Regex + PDF** لاستخراج الأقسام والجمل في `pdf_to_csv.py`.
- **أدوات استخراج انتقائية** للتحقق من المستوى والقسم والصفحة والجملة في `language_extraction.py`.
- **طبقة طلبات OpenAI** (`openai_request.py`) مع البحث في الكاش، ومعالجة الـ prompt، ومحاولات إعادة الاستخلاص اليدوي لـ JSON. 
- **خط عرض متعدد اللغات** مقدم عبر `app.py` و`templates/index.html`.
- **تطبيع فونتيكية يابانية** يحوّل بيانات katakana إلى hiragana قبل العرض.
- **تخزين مؤقت على القرص** ضمن `cache/` لطلبات/استجابات الترجمة المولّدة.

## 🗂️ بنية المشروع

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

## ✅ المتطلبات المسبقة

- Python `3.10+`
- `pip` مع بيئة افتراضية نشطة (`venv` مستحسن)
- مفتاح OpenAI API (`OPENAI_API_KEY`) عند استخدام التوليد عبر الذكاء الاصطناعي
- اتصال إنترنت فعّال لتنزيل ملفات PDF وطلبات OpenAI

نظرًا لعدم وجود ملف lock في هذا المستودع، تُستنتج التبعيات من الواردات والمحتوى السابق:

| الحزمة | مستخدم بواسطة |
|---|---|
| `tornado` | `app.py` |
| `openai` | `openai_request.py`, `multilingual_sentence.py` |
| `PyPDF2` | `pdf_to_csv.py`, `language_extraction.py` |
| `requests` | `download_course_text.py` |
| `beautifulsoup4` | `download_course_text.py` |

## 🛠️ التثبيت

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

| تلميحة الإعداد | الأمر |
|---|---|
| تفعيل البيئة الافتراضية | `source .venv/bin/activate` |
| إعادة إنشاء البيئة | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| تشغيل الفحوص | `python -m pip check` |

## 🚀 الاستخدام

شغّل السكربتات بهذا الترتيب للحصول على مسار العمل القياسي:

### 1) تنزيل ملفات PDF المصدر

```bash
python download_course_text.py
```

ينزل الملفات إلى `downloaded_pdfs/`.

### 2) استخراج محتوى PDF إلى CSV

```bash
python pdf_to_csv.py
```

ينتج افتراضيًا الملف `japanese_language_data.csv`.

### 3) فحص مقطع PDF محدد (اختياري)

```bash
python language_extraction.py
```

مفيد للتحقق من مسارات `level` و`section` و`page` و`sentence_num` المحددة قبل إنشاء مجموعات بيانات أوسع.

### 4) بناء حمولات الجمل متعددة اللغات (اختياري)

```bash
python multilingual_sentence.py
```

ملاحظات سلوك حالية لتحسين الموثوقية:

- المعالجة الحالية تتعامل مع الصف الأول فقط بسبب وجود `break` مبكر.
- بناء الـ prompt يشير إلى `japanese_text`، وهي متغير حالية غير متناسقة مع متغير صف CSV المستخرج وقد تفشل العملية.

### 5) تشغيل تطبيق الويب

```bash
python app.py
```

- منفذ Tornado الافتراضي: `7788`
- الرابط: `http://localhost:7788/`
- وجود اختلاف يجب التحقق منه في السجلات: رسالة الإقلاع الحالية تشير إلى `http://localhost:8888`.

## ⚙️ الإعدادات

المتغيرات البيئية المتوقعة من سكربتات زمن التشغيل:

| المتغير | مطلوب | الغرض | الافتراضي |
|---|---|---|---|
| `OPENAI_API_KEY` | نعم (تدفق AI فقط) | مصادقة OpenAI | N/A |
| `OPENAI_MODEL` | لا | تجاوز النموذج في الطلبات | `gpt-4-0125-preview` |

ملفات/مجلدات زمن التشغيل:

- `downloaded_pdfs/` — يملؤه `download_course_text.py`.
- `cache/` — يخزن حمولة prompt/response الخاصة بـ OpenAI المؤقتة.
- `translations.json` — يُستخدم بواسطة واجهة Tornado.
- `templates/index.html` — قالب العرض في المتصفح.

الافتراضات:

- جذر المستودع هو دليل العمل المقصود لجميع السكربتات.
- يمكن إعادة إنشاء ذاكرة الترجمة المؤقتة بأمان إذا كانت قديمة أو مفقودة.

## 🧾 أمثلة

### تنسيق CSV (`japanese_language_data.csv`)

```csv
Level,Unit,Section,Sentence No.,Content
```

### شكل payload ترجمة OpenAI (`translations.json`)

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

### فحص سريع بسيط

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 ملاحظات التطوير

- المشروع غير مُعبّأ (`requirements.txt`، `pyproject.toml`، و CI غير موجودة).
- صُممت السكربتات بطريقة script-first ومقصودة للتحرير وإعادة التشغيل أثناء التكرار.
- تظهر ملفات الدفتر (`*.ipynb`) كأدوات استكشافية ويجب اعتبارها مساعدات بحثية وليست أنابيب إنتاج.
- توجد `i18n/README.*.md` للتوثيق متعدد اللغات، مع وجود كتلة رابط اللغات أعلاه كمدخل موحّد.

## 🩺 استكشاف الأخطاء وإصلاحها

- `ModuleNotFoundError`: ثبّت جميع الحزم المطلوبة في البيئة الافتراضية النشطة.
- خطأ مصادقة OpenAI / استجابات فارغة: تحقق من أن `OPENAI_API_KEY` تم تصديره في shell.
- `FileNotFoundError` الخاص بـ `downloaded_pdfs`: شغّل `python download_course_text.py` أولًا.
- مشاكل تحويل OpenAI: تفحص `cache/*.json` وتأكد من صحة صيغة payload المطلوبة من `multilingual_sentence.py`.
- ارتباك عنوان URL للتطبيق: افتح `http://localhost:7788/` بعد التشغيل.

## 🛣️ خارطة الطريق

- إضافة ملف تبعيات (`requirements.txt` أو `pyproject.toml`) للتثبيت القابل للتكرار.
- إزالة `break` أحادية الصف من `multilingual_sentence.py` ودعم توليد كامل الدفعة متعدد اللغات.
- إصلاح استخدام متغير الـ prompt في `multilingual_sentence.py` وإضافة تحقق للإخراج.
- تصحيح سجل عنوان بدء Tornado ليعكس المنفذ `7788`.
- إضافة أعلام CLI (اللغة، المستوى، مسارات المصدر، مسار الإخراج).
- إدخال اختبارات خفيفة لاستخراج PDF، ومنطق إعادة المحاولة/التحليل، والتحقق من مخطط JSON.
- توسيع مستندات المساهمات في متغيرات `i18n` اللغوية.

## 🤝 المساهمة

المساهمات مرحب بها.

1. اعمل `fork` للمستودع.
2. أنشئ فرع ميّزة جديد.
3. قدّم تغييرًا مركّزًا واجعل سير العمل القائم على السكربتات سهل الإعادة.
4. افتح طلب دمج مع توضيح واضح للأسباب وملاحظات السلوك قبل/بعد.

إذا حدّثت منطق الاستخراج، أضف نماذج مدخلات ومخرجات في وصف طلب الدمج.

## 🙏 الشكر والتقدير

- روابط محتوى دورات Rosetta Stone في `rs_html.py` هي مصدر مراجع PDF القابلة للتنزيل.
- تُستخدم واجهات OpenAI APIs للتوليد متعدد اللغات وتجارب التعليق الفونيتيكي.



## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
