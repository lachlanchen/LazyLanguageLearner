[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# lazylanguagelearner


[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

Lerne Sprachen auf eine faule Art.

## 🌍 Überblick

LazyLanguageLearner ist ein Python-basierter Sprachlern-Workflow, der Folgendes kombiniert:

- PDF-Beschaffung für Rosetta-Stone-Kursinhaltsdokumente.
- PDF-Parsing und Satzextraktion in CSV-Datensätze.
- OpenAI-gestützte mehrsprachige Satzkonvertierung mit Phonetik-Paaren und lokalem Festplatten-Cache.
- Eine schlanke Tornado-Web-App, die mehrsprachigen Text mit Ruby-Phonetikannotationen rendert.

Das aktuelle Repository ist skriptgetrieben (noch nicht als Pip-Modul paketiert), mit Datendateien und Notebooks direkt im Repo.

## ✨ Funktionen

- Lädt Sprachkurs-PDFs aus Links herunter, die in `rs_html.py` eingebettet sind (`download_course_text.py`).
- Extrahiert Abschnitts-/Satzdaten aus PDFs in strukturierte CSV (`pdf_to_csv.py`, `language_extraction.py`).
- Cached OpenAI-Prompt-/Response-Daten in `cache/*.json`, um wiederholte API-Nutzung zu reduzieren (`openai_request.py`).
- Parst KI-Antworten in JSON mit Retry-Logik und benutzerdefinierten JSON-Parsing-Fehlern.
- Stellt mehrsprachige Satzblöcke aus `translations.json` über Tornado bereit (`app.py` + `templates/index.html`).
- Enthält japanische Phonetik-Normalisierung (`katakana` zu `hiragana`) vor dem Rendering.

## 🗂️ Projektstruktur

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
│   └── (derzeit leer)
└── *.ipynb
```

## ✅ Voraussetzungen

Annahmen (da derzeit kein Lockfile oder Dependency-Manifest eingecheckt ist):

- Python 3.10+ (funktioniert wahrscheinlich auch auf nahegelegenen Versionen; exakte getestete Matrix ist nicht deklariert).
- `pip` und `venv`.
- OpenAI-API-Key für modellgestützte Skripte.

Aus Imports abgeleitete Python-Abhängigkeiten:

| Package | Verwendet von |
|---|---|
| `tornado` | Webserver in `app.py` |
| `openai` | API-Aufrufe in `openai_request.py` |
| `PyPDF2` | PDF-Parsing in Extraktionsskripten |
| `requests` | PDF-Downloads in `download_course_text.py` |
| `beautifulsoup4` | HTML-Parsing im Downloader |

## 🛠️ Installation

```bash
# from repository root
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

## 🚀 Verwendung

### 1) Quell-PDFs herunterladen

```bash
python download_course_text.py
```

Dies erstellt `downloaded_pdfs/` und speichert dort Sprach-/Unit-PDFs.

### 2) Japanische PDF-Inhalte nach CSV extrahieren

```bash
python pdf_to_csv.py
```

Standardausgabe des aktuellen Skripts: `japanese_language_data.csv`.

### 3) (Optional) Abschnitts-/Seiten-/Satztext interaktiv zuschneiden

```bash
python language_extraction.py
```

Das Skript enthält bearbeitbare Beispielvariablen (`level`, `section`, `sentence_num`) und gibt extrahierten Text aus.

### 4) Mehrsprachiges JSON mit OpenAI-Flow generieren

```bash
python multilingual_sentence.py
```

Hinweise zum aktuellen Verhalten:

- Es wird nur die erste CSV-Zeile verarbeitet (`break` in der Schleife).
- Das Skript referenziert derzeit eine undefinierte Variable (`japanese_text`) bei der Prompt-Erstellung und braucht daher vor zuverlässiger Nutzung einen kleinen Fix.

### 5) Die Web-App starten

```bash
python app.py
```

- Tornado lauscht auf Port `7788`.
- Im Browser öffnen: `http://localhost:7788/`.
- Hinweis: Das Start-Log gibt derzeit `http://localhost:8888` aus, obwohl gebunden auf `7788`.

## ⚙️ Konfiguration

Umgebungsvariablen:

| Variable | Erforderlich | Zweck | Aktueller Standard |
|---|---|---|---|
| `OPENAI_API_KEY` | Ja | Erforderlich für `OpenAI()`-Client-Initialisierung | N/A |
| `OPENAI_MODEL` | Nein | Optionales Override für das Chat-Modell | `gpt-4-0125-preview` |

Laufzeitdateien/-verzeichnisse:

- `downloaded_pdfs/`: vom Downloader erstellt, von Extraktionsskripten verwendet.
- `cache/`: Request-/Response-Cache für OpenAI-Aufrufe.
- `translations.json`: Datenquelle für Tornado-UI-Rendering.

## 🧾 Datenformat-Beispiele

### CSV (`japanese_language_data.csv`)

Von `pdf_to_csv.py` verwendeter Header:

```csv
Level,Unit,Section,Sentence No.,Content
```

### JSON (`translations.json`)

Die Web-UI erwartet Sprachschlüssel mit `pairs`-Einträgen, die `part` und `phonetic` enthalten:

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

## 🧪 Entwicklungshinweise

- Dieses Repo hat derzeit kein `requirements.txt`, `pyproject.toml` oder CI-Workflow.
- Skripte sind für die direkte Ausführung vom Repository-Root ausgelegt.
- Bestehende Notebooks (`*.ipynb`) wirken explorativ/prototyping-orientiert.
- Große CSV-Artefakte sind direkt in Git versioniert.
- `i18n/` existiert und ist bereit für übersetzte README-Varianten.

## 🩺 Fehlerbehebung

- `ModuleNotFoundError`: aus Imports abgeleitete Abhängigkeiten in der aktiven virtuellen Umgebung installieren.
- `OPENAI`-Auth-Fehler: sicherstellen, dass `OPENAI_API_KEY` in der Shell exportiert ist.
- `FileNotFoundError: downloaded_pdfs`: zuerst `python download_course_text.py` ausführen.
- `multilingual_sentence.py`-Fehler bei `japanese_text`: `japanese_text` in der Prompt-Konstruktion durch `content` ersetzen.
- Verwirrung um App-Port: `http://localhost:7788/` verwenden, sofern `app.listen(...)` nicht geändert wird.

## 🛣️ Roadmap

Mögliche nächste Schritte für das Projekt:

- Dependency-Manifest hinzufügen (`requirements.txt` oder `pyproject.toml`).
- Prompt-Variable in `multilingual_sentence.py` korrigieren und den Ein-Zeilen-`break` für Batch-Verarbeitung entfernen.
- Tornado-Startup-Print-URL mit gebundenem Port angleichen.
- Tests für PDF-Extraktions-Regex-Verhalten und JSON-Parsing-/Retry-Logik hinzufügen.
- CLI-Argumente für Sprache/Level/Pfade hinzufügen, um In-File-Bearbeitungen zu reduzieren.
- `i18n/` mit übersetzten README-Dateien füllen.

## 🤝 Beitragen

Beiträge sind willkommen.

1. Repository forken.
2. Feature-Branch erstellen.
3. Fokussierte Änderungen mit klaren Commit-Messages vornehmen.
4. Pull Request öffnen, der beschreibt, was geändert wurde und warum.

Wenn du die Extraktionslogik änderst, füge Beispiel-Input/Output-Snippets hinzu, damit das Review einfacher wird.

## 🙏 Danksagungen

- Rosetta-Stone-Kursinhaltslinks sind in `rs_html.py` eingebettet und werden als Quellreferenzen für PDF-Downloads verwendet.
- Die OpenAI API wird für mehrsprachige Generierung und Phonetik-Strukturierung genutzt.

## 📄 Lizenz

Dieses Projekt ist unter der Apache License 2.0 lizenziert. Siehe [LICENSE](LICENSE).
