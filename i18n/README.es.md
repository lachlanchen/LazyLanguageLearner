[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# lazylanguagelearner


[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](../LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

Aprende idiomas de forma perezosa.

## 🌍 Resumen

LazyLanguageLearner es un flujo de aprendizaje de idiomas basado en Python que combina:

- Adquisición de PDFs para documentos de contenido de cursos de Rosetta Stone.
- Análisis de PDFs y extracción de oraciones en datasets CSV.
- Conversión multilingüe de oraciones con OpenAI, con pares fonéticos y caché local en disco.
- Una app web ligera con Tornado que renderiza texto multilingüe con anotaciones fonéticas tipo ruby.

Actualmente, el repositorio está orientado a scripts (todavía no está empaquetado como módulo `pip`), con archivos de datos y notebooks incluidos directamente en el repositorio.

## ✨ Funcionalidades

- Descarga PDFs de cursos de idiomas desde enlaces embebidos en `rs_html.py` (`download_course_text.py`).
- Extrae datos de secciones/oraciones desde PDFs a CSV estructurado (`pdf_to_csv.py`, `language_extraction.py`).
- Almacena en caché datos de prompt/respuesta de OpenAI en `cache/*.json` para reducir uso repetido de la API (`openai_request.py`).
- Analiza respuestas de IA en JSON con lógica de reintentos y errores personalizados de parseo JSON.
- Sirve bloques de oraciones multilingües desde `translations.json` mediante Tornado (`app.py` + `templates/index.html`).
- Incluye normalización fonética japonesa (`katakana` a `hiragana`) antes del renderizado.

## 🗂️ Estructura del Proyecto

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
│   └── (contains translated READMEs)
└── *.ipynb
```

## ✅ Requisitos Previos

Supuestos (porque actualmente no hay lockfile ni manifiesto de dependencias versionado):

- Python 3.10+ (probablemente funcione en versiones cercanas; la matriz exacta probada no está declarada).
- `pip` y `venv`.
- Clave de API de OpenAI para los scripts basados en modelos.

Dependencias de Python inferidas desde los imports:

| Paquete | Usado por |
|---|---|
| `tornado` | Servidor web en `app.py` |
| `openai` | Llamadas API en `openai_request.py` |
| `PyPDF2` | Parseo de PDF en scripts de extracción |
| `requests` | Descargas de PDF en `download_course_text.py` |
| `beautifulsoup4` | Parseo de HTML en el descargador |

## 🛠️ Instalación

```bash
# from repository root
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

## 🚀 Uso

### 1) Descargar PDFs fuente

```bash
python download_course_text.py
```

Esto crea `downloaded_pdfs/` y guarda allí los PDFs por idioma/unidad.

### 2) Extraer contenido del PDF en japonés a CSV

```bash
python pdf_to_csv.py
```

Salida por defecto del script actual: `japanese_language_data.csv`.

### 3) (Opcional) Recortar texto por sección/página/oración de forma interactiva

```bash
python language_extraction.py
```

El script contiene variables de ejemplo editables (`level`, `section`, `sentence_num`) e imprime el texto extraído.

### 4) Generar JSON multilingüe usando el flujo de OpenAI

```bash
python multilingual_sentence.py
```

Notas sobre el comportamiento actual:

- Solo se procesa la primera fila del CSV (`break` en el bucle).
- El script actualmente referencia una variable no definida (`japanese_text`) en la creación del prompt, así que necesita un ajuste pequeño antes de un uso fiable.

### 5) Ejecutar la app web

```bash
python app.py
```

- Tornado escucha en el puerto `7788`.
- Abrir en el navegador: `http://localhost:7788/`.
- Nota: en el log de inicio actualmente se imprime `http://localhost:8888` aunque el bind es `7788`.

## ⚙️ Configuración

Variables de entorno:

| Variable | Requerida | Propósito | Valor por defecto actual |
|---|---|---|---|
| `OPENAI_API_KEY` | Sí | Requerida para la inicialización del cliente `OpenAI()` | N/A |
| `OPENAI_MODEL` | No | Sobrescritura opcional del modelo de chat | `gpt-4-0125-preview` |

Archivos/directorios en ejecución:

- `downloaded_pdfs/`: creado por el downloader, usado por scripts de extracción.
- `cache/`: caché de solicitudes/respuestas para llamadas a OpenAI.
- `translations.json`: fuente de datos para el renderizado de la UI de Tornado.

## 🧾 Ejemplos de Formato de Datos

### CSV (`japanese_language_data.csv`)

Encabezado usado por `pdf_to_csv.py`:

```csv
Level,Unit,Section,Sentence No.,Content
```

### JSON (`translations.json`)

La UI web espera claves de idioma con entradas `pairs` que contienen `part` y `phonetic`:

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

## 🧪 Notas de Desarrollo

- Este repositorio actualmente no tiene `requirements.txt`, `pyproject.toml` ni workflow de CI.
- Los scripts están diseñados para ejecutarse directamente desde la raíz del repositorio.
- Los notebooks existentes (`*.ipynb`) parecen orientados a exploración/prototipado.
- Los artefactos CSV grandes están versionados directamente en Git.
- `i18n/` existe y está listo para variantes traducidas del README.

## 🩺 Solución de Problemas

- `ModuleNotFoundError`: instala las dependencias inferidas en el entorno virtual activo.
- Errores de autenticación de `OPENAI`: verifica que `OPENAI_API_KEY` esté exportada en tu shell.
- `FileNotFoundError: downloaded_pdfs`: ejecuta primero `python download_course_text.py`.
- Falla en `multilingual_sentence.py` por `japanese_text`: reemplaza `japanese_text` por `content` en la construcción del prompt.
- Confusión con puertos de la app: usa `http://localhost:7788/` a menos que cambie `app.listen(...)`.

## 🛣️ Hoja de Ruta

Siguientes pasos potenciales para el proyecto:

- Añadir manifiesto de dependencias (`requirements.txt` o `pyproject.toml`).
- Corregir la variable del prompt en `multilingual_sentence.py` y eliminar el `break` de una sola fila para procesamiento por lotes.
- Alinear la URL impresa al iniciar Tornado con el puerto realmente enlazado.
- Agregar pruebas para el comportamiento de regex en extracción de PDF y para la lógica de parseo JSON/reintentos.
- Agregar argumentos CLI para idioma/nivel/rutas y así reducir ediciones dentro del código.
- Completar `i18n/` con archivos README traducidos.

## 🤝 Contribuciones

Las contribuciones son bienvenidas.

1. Haz un fork del repositorio.
2. Crea una rama de funcionalidad.
3. Realiza cambios enfocados con mensajes de commit claros.
4. Abre un pull request describiendo qué cambió y por qué.

Si modificas la lógica de extracción, incluye ejemplos de entrada/salida para facilitar la revisión.

## 🙏 Agradecimientos

- Los enlaces de contenido de cursos de Rosetta Stone están embebidos en `rs_html.py` y se usan como referencias de origen para descargar PDFs.
- La API de OpenAI se utiliza para la generación multilingüe y la estructuración fonética.

## 📄 Licencia

Este proyecto está licenciado bajo Apache License 2.0. Consulta [LICENSE](../LICENSE).
