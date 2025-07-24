# pdf-toc
Table of Contents for PDF files of scientific papers
# 🧠 Génération de sommaires à partir de documents scientifiques (PDF)

Ce projet s'inscrit dans le cadre d'une étude de cas, visant à explorer différentes approches pour extraire automatiquement des **sommaires** (et éventuellement des résumés de sections) à partir d'articles scientifiques disponibles sur [arXiv.org](https://arxiv.org/list/cs.AI/recent), au format PDF.

L'objectif est de démontrer une **capacité à structurer une réflexion**, à analyser les résultats obtenus et à proposer des solutions pertinentes.

---

## 🎯 Objectif

- Générer automatiquement un **sommaire structuré** à partir d’un PDF.
- (Optionnel) Générer un court **résumé pour chaque section**.
- Explorer différentes **approches heuristiques et basées sur des modèles de langage**.
- Identifier les limites de chaque solution pour justifier les choix techniques.

---

## 🧠 Démarche

### 1. Compréhension du problème
- Identifier les sections, sous-sections et hiérarchies dans les articles scientifiques.
- Traiter la diversité des formats de titres (ex. : `1 Introduction`, `2.1 Travaux connexes`, titres en majuscules, etc.).
- Traiter la diversité des formats de documents PDF (double colonne, police, etc.)
- Structuration sous forme de sommaire hiérarchique.

### 2. Veille technologique
Avant de démarrer le développement, plusieurs outils et bibliothèques ont été explorés pour évaluer leur pertinence dans le cadre de la génération de sommaires à partir de PDF scientifiques :

| Outil / Approche               | Description                                                                 | Statut dans ce projet       | Observations |
|-------------------------------|-----------------------------------------------------------------------------|------------------------------|--------------|
| **GROBID**                    | Extraction structurée des métadonnées et sections (Java + Docker)           | ❌ Non utilisé                | Installation échouée malgré plusieurs tentatives. Très adapté sur le fond. |
| **LangChain + Unstructured**  | Chargement de PDF en blocs logiques (titres, paragraphes, tableaux, etc.)  | ❌ Trop instable             | Erreurs multiples liées aux dépendances. Intéressant mais fragile. |
| **LayoutParser**              | Analyse visuelle des layouts (via OCR et détection de blocs)                | ⚠️ Partiellement adapté      | Mieux adapté aux documents visuels qu’aux textes structurés. |
| **LLMs Hugging Face**         | Utilisation de modèles comme BART, T5 pour classifier ou générer des TOC    | ✅ Utilisé à titre comparatif | Faciles à intégrer, résultats variables, structure parfois imprécise. |
| **GPT-4o (OpenAI)**           | Extraction de TOC par prompt sur chaque page avec structuration JSON        | ✅ Méthode principale retenue | Résultats fiables, hiérarchie respectée, bon compromis simplicité/précision. |


### 3. Implémentation par itérations

#### 🛠 Approche 1 : Extraction heuristique

##### 🔧 Implémentation
- Extraction du texte via `pdfplumber` ou `PyMuPDF`.
- Détection de titres par regex :
  - Numérotation (`1.`, `1.1.`, "I", "X.")
  - Majuscules
  - Début de ligne avec majuscule
  - Fin de ligne sans ponctuation
- Structuration hiérarchique des titres.
##### ⚠️ Limites
- Sensible à la diversité des formats PDF (mise en page, colonnes, polices...).
- Trop de règles (regex) risquent d’éliminer de vrais titres ; trop peu en détectent de faux.
- Ne permet pas de capturer tous les styles ou formats de titres possibles.
- Confond souvent les titres avec des listes numérotées.
- Ne tient pas compte des styles typographiques (gras, taille, indentation).
- Ne gère pas les documents multilingues ou contenant des symboles atypiques.

#### 🤖 Approche 2 : Utilisation de modèles de langage (LLM)
##### 🔧 Implémentation
- Lecture du PDF **page par page** via `PyMuPDF` pour réduire la taille des prompts.
- Envoi de chaque page à un **LLM** (ex. `GPT-4o`) via l’API OpenAI.
- Génération directe d’un JSON structuré contenant les titres de sections détectés.
- Nettoyage des réponses : filtrage du format, validation JSON, gestion des erreurs.
- Sauvegarde intermédiaire après chaque page pour éviter la perte de données.

##### ⚠️ Limites
- **Coût et quota API** : les appels multiples à l’API GPT-4 peuvent dépasser les quotas (TPM), notamment sur des documents longs.
- **Longueur des prompts** : certains contenus dépassent les limites autorisées (~30k tokens/min pour GPT-4o), ce qui peut provoquer des erreurs 429.
- **Variabilité des réponses** : les formats retournés par les LLM ne sont pas toujours strictement conformes au format attendu → besoin de post-traitement.
- **Latence cumulée** : traitement long lorsqu’on parcourt plusieurs documents page par page, même avec des `sleep()` optimisés.



#### 🔬 Approche 3 : Classification / BERT
##### 🔧 Implémentation
- Classifier chaque ligne comme "titre de section" ou "texte normal".
- Possibilité d’intégrer LayoutLM ou SciBERT pour améliorer la précision.

##### ⚠️ Limites
---

## 📊 Évaluation & perspectives

- Chaque méthode a été testée sur des exemples réels (arXiv).
- Les résultats ont été analysés en termes de :
  - Précision des titres détectés
  - Structuration hiérarchique
  - Résilience face aux mises en page variées

### 🔭 Améliorations envisagées
- Intégration d’un **graphe de connaissances** pour représenter les relations entre sections, et la structure complète du document.
- Fine-tuning d’un modèle sur un corpus d’articles scientifiques.
- Génération automatique de résumés par section.
- Interface utilisateur ou pipeline RAG.

---

## 📁 Arborescence du projet
```bash
├── notebook.ipynb         # Notebook principal avec l'ensemble des expérimentations
├── README.md              # Documentation du projet
├── .env                   # Fichier de configuration (clé API, ignoré par Git)
├── articles/              # Dossier contenant les PDF extraits de arXiv
├── output_v1/             # Résultats de l'approche 1 (regex + heuristiques)
├── output_v2/             # Résultats de l'approche 2 (LLM, GPT-4o)
└── requirements.txt       # Liste des dépendances
```