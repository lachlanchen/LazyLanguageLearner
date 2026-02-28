[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# lazylanguagelearner

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Script--Driven-orange)
![Web](https://img.shields.io/badge/Web-Tornado-5C2D91?logo=tornado)
![AI](https://img.shields.io/badge/OpenAI-API-10A37F?logo=openai&logoColor=white)

Apprenez une langue de manière paresseuse.

## 🌍 Vue d'ensemble

LazyLanguageLearner est un workflow d'apprentissage des langues basé sur Python qui combine :

- L'acquisition de PDF pour les documents de contenu de cours Rosetta Stone.
- L'analyse de PDF et l'extraction de phrases vers des jeux de données CSV.
- La conversion multilingue de phrases via OpenAI, avec paires phonétiques et cache local sur disque.
- Une application web Tornado légère qui affiche du texte multilingue avec des annotations phonétiques ruby.

Le dépôt actuel est piloté par scripts (pas encore empaqueté comme module pip), avec fichiers de données et notebooks inclus directement dans le dépôt.

## ✨ Fonctionnalités

- Télécharge les PDF de cours de langues à partir de liens intégrés dans `rs_html.py` (`download_course_text.py`).
- Extrait les données de sections/phrases des PDF vers un CSV structuré (`pdf_to_csv.py`, `language_extraction.py`).
- Met en cache les données prompt/réponse OpenAI dans `cache/*.json` pour réduire l'usage API répété (`openai_request.py`).
- Analyse les réponses IA en JSON avec logique de nouvelle tentative et erreurs d'analyse JSON personnalisées.
- Sert des blocs de phrases multilingues depuis `translations.json` via Tornado (`app.py` + `templates/index.html`).
- Inclut la normalisation phonétique japonaise (`katakana` vers `hiragana`) avant affichage.

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
│   └── (currently empty)
└── *.ipynb
```

## ✅ Prérequis

Hypothèses (car aucun lockfile ou manifeste de dépendances n'est actuellement versionné) :

- Python 3.10+ (fonctionne probablement sur des versions proches ; la matrice testée exacte n'est pas déclarée).
- `pip` et `venv`.
- Clé API OpenAI pour les scripts reposant sur un modèle.

Dépendances Python déduites des imports :

| Package | Utilisé par |
|---|---|
| `tornado` | Serveur web dans `app.py` |
| `openai` | Appels API dans `openai_request.py` |
| `PyPDF2` | Analyse PDF dans les scripts d'extraction |
| `requests` | Téléchargements PDF dans `download_course_text.py` |
| `beautifulsoup4` | Analyse HTML dans le téléchargeur |

## 🛠️ Installation

```bash
# from repository root
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install tornado openai PyPDF2 requests beautifulsoup4
```

## 🚀 Utilisation

### 1) Télécharger les PDF sources

```bash
python download_course_text.py
```

Cela crée `downloaded_pdfs/` et y enregistre les PDF langue/unité.

### 2) Extraire le contenu PDF japonais vers CSV

```bash
python pdf_to_csv.py
```

Sortie par défaut du script actuel : `japanese_language_data.csv`.

### 3) (Optionnel) Découper le texte section/page/phrase de manière interactive

```bash
python language_extraction.py
```

Le script contient des variables d'exemple modifiables (`level`, `section`, `sentence_num`) et affiche le texte extrait.

### 4) Générer un JSON multilingue via le flux OpenAI

```bash
python multilingual_sentence.py
```

Remarques sur le comportement actuel :

- Seule la première ligne du CSV est traitée (`break` dans la boucle).
- Le script référence actuellement une variable non définie (`japanese_text`) lors de la création du prompt ; une petite correction est nécessaire avant un usage fiable.

### 5) Lancer l'application web

```bash
python app.py
```

- Tornado écoute sur le port `7788`.
- Ouvrir dans le navigateur : `http://localhost:7788/`.
- Remarque : le log de démarrage affiche actuellement `http://localhost:8888` alors que le binding est `7788`.

## ⚙️ Configuration

Variables d'environnement :

| Variable | Requise | Objectif | Valeur par défaut actuelle |
|---|---|---|---|
| `OPENAI_API_KEY` | Oui | Requise par l'initialisation du client `OpenAI()` | N/A |
| `OPENAI_MODEL` | Non | Surcharge optionnelle du modèle de chat | `gpt-4-0125-preview` |

Fichiers/répertoires d'exécution :

- `downloaded_pdfs/` : créé par le téléchargeur, utilisé par les scripts d'extraction.
- `cache/` : cache des requêtes/réponses pour les appels OpenAI.
- `translations.json` : source de données pour le rendu de l'interface Tornado.

## 🧾 Exemples de format de données

### CSV (`japanese_language_data.csv`)

En-tête utilisé par `pdf_to_csv.py` :

```csv
Level,Unit,Section,Sentence No.,Content
```

### JSON (`translations.json`)

L'interface web attend des clés de langue avec des entrées `pairs` contenant `part` et `phonetic` :

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

## 🧪 Notes de développement

- Ce dépôt n'a actuellement ni `requirements.txt`, ni `pyproject.toml`, ni workflow CI.
- Les scripts sont conçus pour une exécution directe depuis la racine du dépôt.
- Les notebooks existants (`*.ipynb`) semblent orientés exploration/prototypage.
- Les artefacts CSV volumineux sont versionnés directement dans Git.
- `i18n/` existe et est prêt pour les variantes README traduites.

## 🩺 Dépannage

- `ModuleNotFoundError` : installez les dépendances déduites dans l'environnement virtuel actif.
- Erreurs d'auth `OPENAI` : vérifiez que `OPENAI_API_KEY` est exportée dans le shell.
- `FileNotFoundError: downloaded_pdfs` : lancez d'abord `python download_course_text.py`.
- Échec de `multilingual_sentence.py` sur `japanese_text` : remplacez `japanese_text` par `content` dans la construction du prompt.
- Confusion sur le port de l'application : utilisez `http://localhost:7788/` sauf si `app.listen(...)` est modifié.

## 🛣️ Feuille de route

Prochaines étapes potentielles pour le projet :

- Ajouter un manifeste de dépendances (`requirements.txt` ou `pyproject.toml`).
- Corriger la variable du prompt dans `multilingual_sentence.py` et supprimer le `break` mono-ligne pour un traitement par lot.
- Aligner l'URL affichée au démarrage Tornado avec le port réellement bindé.
- Ajouter des tests pour le comportement regex d'extraction PDF et la logique d'analyse JSON/retry.
- Ajouter des arguments CLI pour langue/niveau/chemins afin de réduire les modifications dans le code.
- Remplir `i18n/` avec les fichiers README traduits.

## 🤝 Contribution

Les contributions sont les bienvenues.

1. Forkez le dépôt.
2. Créez une branche de fonctionnalité.
3. Faites des changements ciblés avec des messages de commit clairs.
4. Ouvrez une pull request décrivant ce qui a changé et pourquoi.

Si vous modifiez la logique d'extraction, incluez des exemples d'entrée/sortie pour faciliter la revue.

## 🙏 Remerciements

- Les liens de contenu de cours Rosetta Stone sont intégrés dans `rs_html.py` et utilisés comme références sources pour les téléchargements PDF.
- L'API OpenAI est utilisée pour la génération multilingue et la structuration phonétique.

## 📄 Licence

Ce projet est sous licence Apache License 2.0. Voir [LICENSE](LICENSE).
