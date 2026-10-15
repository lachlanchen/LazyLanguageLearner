[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner — apps and four buddies](https://lazying.art/media/brand/lazyingart-intro.gif)](https://lazying.art)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| Atributo | Valor |
|---|---|
| Tipo | Flujo de aprendizaje de idiomas multilingüe guiado por scripts |
| Entorno de ejecución | Python CLI + aplicación web Tornado |
| Fuente principal | PDFs del curso de Rosetta Stone |
| Almacenamiento | Caché local en CSV + JSON |
| Puerto predeterminado | `7788` |

LazyLanguageLearner es un flujo de trabajo en Python orientado a scripts para convertir PDFs de cursos de idiomas en datos de aprendizaje multilingües reutilizables y renderizarlos en una interfaz web mínima.

## 🌍 Visión general

El repositorio combina extracción, transformación y entrega de contenido:

| Paso | Objetivo |
|---|---|
| 1 | Descarga PDFs de contenido de cursos de Rosetta Stone desde enlaces incrustados en `rs_html.py`. |
| 2 | Analiza los PDFs en filas de CSV a nivel de frase para su transformación. |
| 3 | Genera variantes multilingües/fonéticas mediante OpenAI con caché en disco. |
| 4 | Renderiza las frases estructuradas en una interfaz web Tornado con anotaciones fonéticas. |

Este proyecto está pensado para ser intencionalmente liviano y centrado en la raíz del repositorio: los scripts están diseñados para ejecutarse directamente desde la raíz del proyecto en lugar de como un paquete instalado.

## ✨ Características

- **Flujo de descarga automatizada** desde enlaces incrustados en `rs_html.py` usando `download_course_text.py`.
- **Pipeline de extracción con expresiones regulares y PDF** para extracción de secciones/frases en `pdf_to_csv.py`.
- **Utilidades de extracción selectiva** para inspección de nivel, sección, página y frase en `language_extraction.py`.
- **Capa de solicitudes OpenAI** (`openai_request.py`) con búsqueda en caché, manejo de prompts y reintentos básicos de extracción de JSON.
- **Pipeline de renderizado multilingüe** servido por `app.py` y `templates/index.html`.
- **Normalización fonética japonesa** que convierte datos en katakana a hiragana antes del renderizado.
- **Caché en disco** en `cache/` para solicitudes y respuestas de traducción generadas.

## 🗂️ Estructura del proyecto

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

## ✅ Requisitos previos

- Python `3.10+`
- `pip` con un entorno virtual activo (`venv` recomendado)
- Una clave de API de OpenAI (`OPENAI_API_KEY`) al usar generación con IA
- Conexión a Internet activa para descargas de PDF y solicitudes a OpenAI

Como este repositorio no incluye archivo de bloqueo, las dependencias se infieren de imports y contenido previo:

| Paquete | Usado por |
|---|---|
| `tornado` | `app.py` |
| `openai` | `openai_request.py`, `multilingual_sentence.py` |
| `PyPDF2` | `pdf_to_csv.py`, `language_extraction.py` |
| `requests` | `download_course_text.py` |
| `beautifulsoup4` | `download_course_text.py` |

## 🛠️ Instalación

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

| Sugerencia de configuración | Comando |
|---|---|
| Activar venv | `source .venv/bin/activate` |
| Recrear entorno | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| Ejecutar comprobaciones | `python -m pip check` |

## 🚀 Uso

Ejecuta los scripts en este orden para el pipeline estándar:

### 1) Descargar PDFs de origen

```bash
python download_course_text.py
```

Descarga PDFs en `downloaded_pdfs/`.

### 2) Extraer contenido del PDF a CSV

```bash
python pdf_to_csv.py
```

Genera `japanese_language_data.csv` de forma predeterminada.

### 3) Inspeccionar un tramo específico del PDF (opcional)

```bash
python language_extraction.py
```

Útil para validar rutas concretas de `level`, `section`, `page` y `sentence_num` antes de generar conjuntos de datos más amplios.

### 4) Construir cargas de frases multilingües (opcional)

```bash
python multilingual_sentence.py
```

Notas de comportamiento actuales para mayor fiabilidad:

- La versión actual procesa solo la primera fila debido a un `break` inicial.
- La generación de prompts hace referencia a `japanese_text`, que actualmente parece inconsistente con la variable de fila CSV extraída y puede fallar.

### 5) Iniciar la aplicación web

```bash
python app.py
```

- Puerto predeterminado de Tornado: `7788`
- URL: `http://localhost:7788/`
- Inconsistencia conocida a verificar en los logs: el mensaje de inicio muestra actualmente `http://localhost:8888`.

## ⚙️ Configuración

Variables de entorno esperadas por los scripts en tiempo de ejecución:

| Variable | Requerida | Propósito | Valor predeterminado |
|---|---|---|---|
| `OPENAI_API_KEY` | Sí (solo flujo de IA) | Autenticación OpenAI | N/A |
| `OPENAI_MODEL` | No | Sobrescritura de modelo en solicitudes | `gpt-4-0125-preview` |

Archivos/directorios de ejecución:

- `downloaded_pdfs/` — completado por `download_course_text.py`.
- `cache/` — almacena cachés de solicitud/respuesta de OpenAI.
- `translations.json` — utilizado por la interfaz de Tornado.
- `templates/index.html` — plantilla de renderizado en navegador.

Suposiciones:

- La raíz del repositorio es el directorio de trabajo previsto para todos los scripts.
- La caché de traducción puede regenerarse de forma segura si está obsoleta o falta.

## 🧾 Ejemplos

### Formato CSV (`japanese_language_data.csv`)

```csv
Level,Unit,Section,Sentence No.,Content
```

### Forma de carga de traducción de OpenAI (`translations.json`)

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

### Verificación rápida mínima

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 Notas de desarrollo

- El proyecto no está empaquetado (`requirements.txt`, `pyproject.toml` y CI no están presentes).
- Los scripts están orientados a scripts y se espera que se editen y vuelvan a ejecutarse durante la iteración.
- Los archivos de notebook parecen exploratorios y deben tratarse como ayudas de investigación, no como pipelines de producción.
- `i18n/README.*.md` ya existe para documentación multilingüe, siendo este archivo el punto de entrada compartido con el bloque de navegación superior.

## 🩺 Solución de problemas

- `ModuleNotFoundError`: instala todos los paquetes requeridos en el entorno virtual activo.
- Error de autenticación de `OPENAI` / respuestas vacías: verifica que `OPENAI_API_KEY` esté exportada en tu terminal.
- `FileNotFoundError` para `downloaded_pdfs`: ejecuta primero `python download_course_text.py`.
- Problemas con la conversión de OpenAI: revisa `cache/*.json` y verifica el formato de carga de entrada esperado por `multilingual_sentence.py`.
- Confusión sobre la URL de la app: abre `http://localhost:7788/` tras iniciar.

## 🛣️ Hoja de ruta

- Añadir un manifiesto de dependencias (`requirements.txt` o `pyproject.toml`) para instalaciones reproducibles.
- Eliminar el `break` de una sola fila de `multilingual_sentence.py` y soportar generación multilingüe por lotes completos.
- Corregir el uso de variables de prompt en `multilingual_sentence.py` y añadir validación de salida.
- Corregir el log de inicio de Tornado para que refleje el puerto `7788`.
- Añadir flags CLI (idioma, nivel, rutas de origen, ruta de salida).
- Introducir pruebas ligeras para extracción, lógica de reintento/análisis y validación de esquema JSON.
- Ampliar la documentación para colaboradores en variantes de `i18n`.

## 🤝 Contribuir

Las contribuciones son bienvenidas.

1. Haz un fork del repositorio.
2. Crea una rama de función.
3. Realiza un cambio enfocado y mantén los flujos de script fáciles de reproducir.
4. Abre una pull request con una justificación clara y notas de comportamiento antes/después.

Si actualizas la lógica de extracción, incluye ejemplos de entrada y salida en la descripción de tu PR.

## 🙏 Agradecimientos

- Los enlaces a contenido de cursos de Rosetta Stone en `rs_html.py` son la fuente de las referencias de corpus PDF descargables.
- Las API de OpenAI se usan para generación multilingüe y experimentos de anotación fonética.



## ❤️ Support

<!-- support:start -->
[![GitHub Sponsors](https://img.shields.io/badge/GitHub-Sponsors-EA4AAA?style=for-the-badge&logo=githubsponsors&logoColor=white)](https://github.com/sponsors/lachlanchen)
[![LazyingArt Support](https://img.shields.io/badge/LazyingArt-Support-0EA5E9?style=for-the-badge)](https://chat.lazying.art/donate)
[![PayPal Support](https://img.shields.io/badge/PayPal-Support-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen)
[![Stripe Support](https://img.shields.io/badge/Stripe-Support-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400)
<!-- support:end -->
