[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| Attribut | Wert |
|---|---|
| Typ | Skriptgesteuerte mehrsprachige Sprachlernpipeline |
| Laufzeit | Python CLI + Tornado-Web-App |
| Hauptquelle | Rosetta Stone Kurs-PDFs |
| Speicher | Lokale CSV- + JSON-Cache-Dateien |
| Standardport | `7788` |

LazyLanguageLearner ist ein skriptbasiertes Python-Workflow für die Umwandlung von Sprachkurs-PDFs in wiederverwendbare mehrsprachige Lerndaten mit der Darstellung in einer minimalistischen Web-Oberfläche.

## 🌍 Überblick

Das Repository verbindet Extraktion, Transformation und Bereitstellung von Inhalten:

| Schritt | Zweck |
|---|---|
| 1 | Lade Rosetta-Stone-Kurs-PDFs von Links aus `rs_html.py`. |
| 2 | Parse PDFs in zeilenweise CSV-Datensätze auf Satzebene zur Weiterverarbeitung. |
| 3 | Erzeuge mehrsprachige/phonetische Varianten über OpenAI mit On-Disk-Caching. |
| 4 | Rendere die strukturierten Sätze mit phonetischer Annotation in einer Tornado-Web-UI. |

Dieses Projekt ist bewusst leichtgewichtig und am Repository-Root verankert: Skripte sollen direkt aus dem Projektstamm ausgeführt werden und nicht als installiertes Paket.

## ✨ Features

- **Automatisierter Download-Workflow** aus eingebetteten Links in `rs_html.py` mit `download_course_text.py`.
- **Regex- + PDF-Extraktionspipeline** zur Sektionen-/Satzextraktion in `pdf_to_csv.py`.
- **Selektive Extraktionswerkzeuge** für Level, Abschnitt, Seite und Satz-Inspektion in `language_extraction.py`.
- **OpenAI-Anfrageschicht** (`openai_request.py`) mit Cache-Lookups, Prompt-Handling und einfachen Wiederholungen bei JSON-Parsing.
- **Sprachübergreifende Rendering-Pipeline** bereitgestellt durch `app.py` und `templates/index.html`.
- **Japanische phonetische Normalisierung**, die Katakana-Daten vor der Anzeige in Hiragana umwandelt.
- **On-Disk-Caching** unter `cache/` für generierte Übersetzungsanfragen und Antworten.

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
│   └── README.*.md
└── *.ipynb
```

## ✅ Voraussetzungen

- Python `3.10+`
- `pip` mit aktivierter virtueller Umgebung (`venv` empfohlen)
- Ein OpenAI-API-Schlüssel (`OPENAI_API_KEY`) für KI-gestützte Generierung
- Eine funktionierende Internetverbindung für PDF-Downloads und OpenAI-Anfragen

Da in diesem Repository kein Lockfile existiert, werden Abhängigkeiten aus Imports und bisherigem Inhalt abgeleitet:

| Paket | Verwendet von |
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

| Setup-Hinweis | Befehl |
|---|---|
| venv aktivieren | `source .venv/bin/activate` |
| Umgebung reproduzieren | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| Checks ausführen | `python -m pip check` |

## 🚀 Nutzung

Führe die Skripte in dieser Reihenfolge für die Standard-Pipeline aus:

### 1) Quell-PDFs herunterladen

```bash
python download_course_text.py
```

Lädt PDFs in `downloaded_pdfs/`.

### 2) PDF-Inhalt in CSV extrahieren

```bash
python pdf_to_csv.py
```

Erzeugt standardmäßig `japanese_language_data.csv`.

### 3) Spezifischen PDF-Ausschnitt prüfen (optional)

```bash
python language_extraction.py
```

Hilfreich zur Validierung spezifischer `level`-, `section`-, `page`- und `sentence_num`-Pfadwerte, bevor größere Datensätze erzeugt werden.

### 4) Mehrsprachige Satz-Payloads erstellen (optional)

```bash
python multilingual_sentence.py
```

Aktuelle Verhaltenshinweise für Zuverlässigkeit:

- Die aktuelle Version verarbeitet wegen eines frühen `break` nur die erste Zeile.
- Die Prompt-Erzeugung verweist auf `japanese_text`, was derzeit inkonsistent mit der extrahierten CSV-Zeilenvariable zu sein scheint und fehlschlagen kann.

### 5) Die Web-App starten

```bash
python app.py
```

- Standard-Tornado-Port: `7788`
- URL: `http://localhost:7788/`
- Bekannte Abweichung in den Logs: Die Startmeldung verweist aktuell auf `http://localhost:8888`.

## ⚙️ Konfiguration

Umgebungsvariablen, die von den Runtime-Skripten erwartet werden:

| Variable | Erforderlich | Zweck | Standard |
|---|---|---|---|
| `OPENAI_API_KEY` | Ja (nur KI-Flow) | OpenAI-Authentifizierung | N/A |
| `OPENAI_MODEL` | Nein | Modell-Override in Anfragen | `gpt-4-0125-preview` |

Runtime-Dateien/-Verzeichnisse:

- `downloaded_pdfs/` — gefüllt durch `download_course_text.py`.
- `cache/` — speichert gecachte OpenAI-Prompt-/Response-Payloads.
- `translations.json` — wird von der Tornado-UI verwendet.
- `templates/index.html` — Browser-Render-Template.

Annahmen:

- Das Repository-Root ist das vorgesehene Arbeitsverzeichnis für alle Skripte.
- Der Übersetzungs-Cache kann bei Veralterung oder Fehlen sicher neu generiert werden.

## 🧾 Beispiele

### CSV-Format (`japanese_language_data.csv`)

```csv
Level,Unit,Section,Sentence No.,Content
```

### OpenAI-Übersetzungspayload-Format (`translations.json`)

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

### Minimaler Schnellcheck

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 Entwicklungshinweise

- Das Projekt ist nicht paketiert (`requirements.txt`, `pyproject.toml` und CI sind nicht vorhanden).
- Skripte sind script-first ausgelegt und sollen während der Iteration bearbeitet und erneut ausgeführt werden.
- Notebook-Dateien sind explorativ und sollten als Forschungshilfen und nicht als Produktionspipelines betrachtet werden.
- `i18n/README.*.md` existiert bereits für mehrsprachige Dokumentation; in dieser Datei dient der Sprach-Navigationsblock auf oberster Ebene als gemeinsamer Einstiegspunkt.

## 🛣️ Roadmap

- Füge ein Abhängigkeitsmanifest (`requirements.txt` oder `pyproject.toml`) für reproduzierbare Installationen hinzu.
- Entferne das Ein-Zeilen-`break` aus `multilingual_sentence.py` und unterstütze die vollständige Batch-Mehrsprachigkeit.
- Korrigiere die Prompt-Variablenverwendung in `multilingual_sentence.py` und füge eine Ausgabevalidierung hinzu.
- Korrigiere das Tornado-Startup-URL-Log auf Port `7788`.
- Ergänze CLI-Flags (Sprache, Level, Quellpfade, Ausgabeziel).
- Führe leichte Tests für Extraktion, Wiederhol-/Parse-Logik und JSON-Schema-Validierung ein.
- Erweitere mitarbeiterorientierte Dokumentation in den Sprachvarianten unter `i18n`.

## 🤝 Beitragen

Beiträge sind willkommen.

1. Forke das Repository.
2. Erstelle einen Feature-Branch.
3. Nimm eine fokussierte Änderung vor und halte die Skript-Workflows gut reproduzierbar.
4. Eröffne einen Pull Request mit klarer Begründung und Vorher-/Nachher-Verhaltensbeschreibung.

Wenn du Extraktionslogik aktualisierst, füge Beispiel-Eingaben und -Ausgaben in deine PR-Beschreibung ein.

## 🙏 Danksagung

- Rosetta Stone Kursinhalts-Links in `rs_html.py` sind die Quelle für die herunterladbaren PDF-Korpus-Referenzen.
- OpenAI-APIs werden für mehrsprachige Generierung und phonetische Annotations-Experimente verwendet.



## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
