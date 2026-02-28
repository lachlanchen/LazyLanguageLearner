[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# LazyLanguageLearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

| Attribut | Valeur |
|---|---|
| Type | Pipeline d'apprentissage multilingue piloté par des scripts |
| Runtime | Application web Tornado + CLI Python |
| Source principale | PDF de cours Rosetta Stone |
| Stockage | Fichiers CSV + cache JSON locaux |
| Port par défaut | `7788` |

LazyLanguageLearner est un flux de travail Python piloté par scripts qui transforme des PDF de cours de langue en données d'apprentissage multilingues réutilisables, puis les affiche dans une interface web minimaliste.

## 🌍 Vue d'ensemble

Le dépôt combine l'extraction, la transformation et la publication du contenu :

| Étape | Objectif |
|---|---|
| 1 | Télécharger les PDF de cours Rosetta Stone depuis les liens intégrés à `rs_html.py`. |
| 2 | Analyser les PDF en lignes CSV de phrase pour la transformation. |
| 3 | Générer des variantes multilingues/phonétiques via OpenAI avec mise en cache disque. |
| 4 | Afficher les phrases structurées dans une interface web Tornado avec annotations phonétiques. |

Ce projet est volontairement léger et centré sur la racine du dépôt : les scripts sont conçus pour être exécutés directement depuis la racine plutôt que comme un package installé.

## ✨ Fonctionnalités

- **Flux de téléchargement automatisé** depuis les liens intégrés dans `rs_html.py` via `download_course_text.py`.
- **Pipeline d'extraction PDF + regex** pour l'extraction de sections et phrases dans `pdf_to_csv.py`.
- **Utilitaires d'extraction sélective** pour le niveau, la section, la page et l'inspection de phrase dans `language_extraction.py`.
- **Couche de requêtes OpenAI** (`openai_request.py`) avec recherche de cache, gestion des prompts et tentatives de réextraction JSON de base.
- **Pipeline de rendu interlangues** servi par `app.py` et `templates/index.html`.
- **Normalisation phonétique japonaise** convertissant les données katakana en hiragana avant rendu.
- **Cache disque** dans `cache/` pour les demandes/réponses de traduction générées.

## 🗂️ Structure du projet

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

## ✅ Prérequis

- Python `3.10+`
- `pip` avec un environnement virtuel actif (`venv` recommandé)
- Une clé API OpenAI (`OPENAI_API_KEY`) pour la génération assistée par IA
- Une connexion Internet fonctionnelle pour les téléchargements PDF et les requêtes OpenAI

Comme ce dépôt ne contient pas de lockfile, les dépendances sont déduites des imports et du contenu existant :

| Package | Utilisé par |
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

| Conseiller de configuration | Commande |
|---|---|
| Activer le venv | `source .venv/bin/activate` |
| Reproduire l'environnement | `pip install tornado openai PyPDF2 requests beautifulsoup4` |
| Exécuter les vérifications | `python -m pip check` |

## 🚀 Utilisation

Exécutez les scripts dans cet ordre pour le pipeline standard :

### 1) Télécharger les PDF source

```bash
python download_course_text.py
```

Télécharge les PDF dans `downloaded_pdfs/`.

### 2) Extraire le contenu des PDF vers CSV

```bash
python pdf_to_csv.py
```

Produit `japanese_language_data.csv` par défaut.

### 3) Inspecter une tranche PDF précise (optionnel)

```bash
python language_extraction.py
```

Utile pour valider des chemins `level`, `section`, `page` et `sentence_num` spécifiques avant de générer des jeux de données plus larges.

### 4) Construire les charges utiles multilingues de phrases (optionnel)

```bash
python multilingual_sentence.py
```

Notes de comportement actuelles pour la fiabilité :

- La version actuelle ne traite que la première ligne en raison d'un `break` précoce.
- La génération de prompts référence `japanese_text`, qui ne correspond actuellement pas toujours à la variable de ligne CSV extraite et peut échouer.

### 5) Lancer l'application web

```bash
python app.py
```

- Port Tornado par défaut : `7788`
- URL : `http://localhost:7788/`
- Écart connu à vérifier dans les journaux : le message de démarrage actuel référence `http://localhost:8888`.

## ⚙️ Configuration

Variables d'environnement attendues par les scripts d'exécution :

| Variable | Requise | Objectif | Valeur par défaut |
|---|---|---|---|
| `OPENAI_API_KEY` | Oui (flux IA uniquement) | Authentification OpenAI | N/A |
| `OPENAI_MODEL` | Non | Remplacement du modèle dans les requêtes | `gpt-4-0125-preview` |

Fichiers/répertoires au runtime :

- `downloaded_pdfs/` — rempli par `download_course_text.py`.
- `cache/` — stocke les charges utiles de requêtes/réponses OpenAI mises en cache.
- `translations.json` — utilisé par l'UI Tornado.
- `templates/index.html` — modèle de rendu navigateur.

Hypothèses :

- La racine du dépôt est le répertoire de travail prévu pour tous les scripts.
- Le cache de traduction peut être régénéré en toute sécurité s'il est périmé ou manquant.

## 🧾 Exemples

### Format CSV (`japanese_language_data.csv`)

```csv
Level,Unit,Section,Sentence No.,Content
```

### Forme de la charge utile de traduction OpenAI (`translations.json`)

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

### Vérification rapide minimale

```bash
python app.py
python - <<'PY'
import json
with open('translations.json', encoding='utf-8') as f:
    print('Loaded', len(json.load(f)), 'language keys')
PY
```

## 🧪 Notes de développement

- Le projet n'est pas packagé (`requirements.txt`, `pyproject.toml` et CI ne sont pas présents).
- Les scripts sont orientés script et destinés à être édités puis relancés pendant l'itération.
- Les fichiers notebook semblent exploratoires et doivent être traités comme des aides de recherche, pas des pipelines de production.
- `i18n/README.*.md` existe déjà pour la documentation multilingue, la barre de navigation langue de ce fichier servant de point d'entrée partagé.

## 🩺 Dépannage

- `ModuleNotFoundError` : installez tous les paquets requis dans l'environnement virtuel actif.
- Erreur d'authentification `OPENAI` / réponses vides : vérifiez que `OPENAI_API_KEY` est exportée dans votre shell.
- `FileNotFoundError` pour `downloaded_pdfs` : exécutez d'abord `python download_course_text.py`.
- Problèmes de conversion OpenAI : inspectez `cache/*.json` et vérifiez le format d'entrée attendu par `multilingual_sentence.py`.
- Confusion sur l'URL de l'app : ouvrez `http://localhost:7788/` après le démarrage.

## 🛣️ Feuille de route

- Ajouter un manifeste de dépendances (`requirements.txt` ou `pyproject.toml`) pour des installations reproductibles.
- Supprimer le `break` de ligne unique dans `multilingual_sentence.py` et prendre en charge la génération multilingue par lots complets.
- Corriger l'utilisation des variables de prompt dans `multilingual_sentence.py` et ajouter une validation des sorties.
- Corriger le journal de démarrage de Tornado pour refléter le port `7788`.
- Ajouter des options CLI (langue, niveau, chemins source, chemin de sortie).
- Introduire des tests légers pour l'extraction, la logique de retry/parse et la validation du schéma JSON.
- Étendre la documentation orientée contributeurs dans les variantes de langue de `i18n`.

## 🤝 Contribuer

Les contributions sont les bienvenues.

1. Forkez le dépôt.
2. Créez une branche de fonctionnalité.
3. Faites un changement ciblé et gardez les workflows de script faciles à reproduire.
4. Ouvrez une pull request avec une justification claire et des notes de comportement avant/après.

Si vous mettez à jour la logique d'extraction, incluez des exemples d'entrées et sorties dans la description de votre PR.

## 🙏 Remerciements

- Les liens de contenu de cours Rosetta Stone dans `rs_html.py` sont la source des références de corpus PDF téléchargeables.
- Les API OpenAI sont utilisées pour la génération multilingue et les expérimentations d'annotations phonétiques.



## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
