# 🧠 Commande `xxxGenQCM`

## Spécification complète – Assistant IA intégré à l’éditeur

### (Compatible Copilot, Cursor, Windsurf, Codeium, et tout moteur de complétion IA)

---

# 📘 1. Objectif général

Cette spécification décrit comment un **assistant IA intégré à l’éditeur** doit exécuter la commande :

```text
xxxGenQCM [param=value] [param=value] …
```

Sa mission :

1. Analyser un **plan de formation Markdown**, même mal structuré
2. Convertir le plan en **YAML structuré (`plan`)**
3. Générer progressivement un **QCM YAML complet**
4. Mémoriser l’état d’avancement (`progress`)
5. Mémoriser les **options initiales** et **les options du dernier run**
6. Permettre de limiter le nombre de questions générées par run
7. Fournir en mode chat un message explicatif synthétique
8. Générer un fichier YAML autosuffisant (`meta + plan + qcm + progress`)
9. **Créer un répertoire `dist` s’il n’existe pas et y écrire le fichier YAML du QCM**.
10. **Respecter la contrainte : nom de fichier ≤ 20 caractères, extension comprise.**

---

# 📝 2. Analyse du fichier Markdown d’entrée

L’assistant doit comprendre des plans de formation variés, même mal formatés.  
Il doit extraire automatiquement :

- le titre de la formation
- les chapitres
- les notions par chapitre
- en ignorant les sections non pertinentes

---

## 2.1. Détection du titre de la formation

Le titre est extrait automatiquement, même si :

- il n’utilise pas la syntaxe `#`
- il est au milieu d’un paragraphe
- il n’est pas explicitement intitulé “Titre”
- il apparaît comme première phrase ou entête du document

Le titre sert à :

- générer `qcm_title` si absent : **“QCM sur <Titre>”**
- générer automatiquement le nom de fichier (≤ 20 caractères extension comprise) :

```
qcm-<slug>.yaml
```

---

## 2.2. Détection du programme

L’assistant doit être capable de détecter des sections intitulées :

- Programme
- Contenu
- Modules
- Curriculum
- Plan
- etc.

Il doit reconnaître :

- des chapitres (`##`, `###`, “Module 1”, “Partie A”, “1.”, etc.)
- des notions :
  - listes à puces
  - phrases séparées
  - concepts séparés par virgules
  - paragraphes courts

Il doit reconstruire un plan structuré :

```yaml
plan:
  chapters:
    - id: module_1_introduction
      title: "Module 1 – Introduction"
      notions:
        - notion 1
        - notion 2
```

---

## 2.3. Sections à ignorer

Ignorer totalement :

- objectifs pédagogiques
- public concerné
- prérequis
- introduction marketing
- méthodologie pédagogique
- logistique
- matériel requis

---

# ⚙️ 3. Paramètres de la commande `xxxGenQCM`

La commande peut apparaître :

- dans un commentaire YAML
- dans un message chat
- insérée dans un fichier existant

Exemples :

```yaml
# xxxGenQCM
# xxxGenQCM questions_per_chapter=40 difficulty=facile
# xxxGenQCM new_questions=12
```

---

## 3.1. Paramètres disponibles

| Paramètre               | Exemple                         | Défaut            | Description                                         |
| ----------------------- | ------------------------------- | ----------------- | --------------------------------------------------- |
| `questions_per_chapter` | `questions_per_chapter=40`      | **20**            | Nombre de questions par chapitre                    |
| `language`              | `language=en`                   | **fr**            | QCM produit en français uniquement                  |
| `difficulty`            | `difficulty=moyen`              | **progressive**   | Difficulté des questions                            |
| `qcm_title`             | `qcm_title="QCM Docker"`        | “QCM sur <Titre>” | Titre du QCM                                        |
| `output_file`           | `output_file="qcm-docker.yaml"` | `qcm-<slug>.yaml` | Nom du fichier (≤ 20 caractères)                    |
| `new_questions`         | `new_questions=15`              | **10**            | Maximum de nouvelles questions générées lors du run |

---

## 3.2. Règles prioritaires

1. Paramètres directement fournis dans la commande
2. Paramètres issus d’un bloc HTML éventuellement présent
3. Valeurs par défaut

---

## 3.3. Règles de génération du slug et du nom de fichier

- Le nom final doit être **≤ 20 caractères extension comprise**
- Format recommandé par défaut :

```
qcm-<slug>.yaml
```

### Construction du slug :

- minuscules
- espaces remplacés par des tirets
- accents supprimés
- mots vides retirés (de, la, le, les, et…)
- tronqué si nécessaire
- sans couper un mot sauf contrainte absolue
- résultat final ≤ 20 caractères avec `qcm-` + slug + `.yaml`

### Si `output_file` est fourni :

- il doit être ajusté **automatiquement** pour respecter la limite de 20 caractères
- seule la partie basename est modifiée
- le fichier est toujours écrit dans `dist/`

---

## 3.4. Répertoire de sortie `dist`

L’assistant doit :

- considérer que le fichier de sortie se trouve dans :

```
dist/<output_file>
```

- si `dist/` n’existe pas, **le créer automatiquement**
- écrire ou mettre à jour le fichier YAML dans ce répertoire

Dans l’en‑tête du fichier :

```
# FILE: dist/<output_file>
```

Dans `meta.output_file` :

- uniquement le nom du fichier (basename), sans `dist/`

---

# 🧾 4. Structure du fichier YAML généré

Le fichier contient **exactement 4 sections** :

```yaml
meta:
plan:
qcm:
progress:
```

---

## 4.1. Section `meta`

Contient :

- `formation_title`
- `qcm_title`
- `output_file`
- `language`
- `difficulty`
- `questions_per_chapter`

### 4.1.1 `meta.options_original`

Snapshot immuable du premier run.

Contient :

- language
- questions_per_chapter
- difficulty
- qcm_title
- output_file
- new_questions

### 4.1.2 `meta.options_last_run`

Mis à jour à chaque run.  
Contient aussi `new_questions`.

---

# 4.2. Section `plan`

Structure exacte :

```yaml
plan:
  chapters:
    - id: "<slug>"
      title: "<Titre>"
      notions:
        - "<notion>"
```

---

# 4.3. Section `qcm`

Structure complète :

```yaml
qcm:
  chapters:
    - id: "<slug>"
      title: "<Titre>"
      questions:
        - id: "<unique>"
          question: "<texte>"
          answers: ["A", "B", "C", "D"]
          correct: <0-3>
          explanation: "<explication>"
```

L’assistant **ajoute** uniquement, jamais ne modifie ni ne supprime.

---

# 4.4. Section `progress`

Permet de reprendre la génération :

```yaml
progress:
  status: "in_progress" | "complete"
  total_chapters: <int>
  completed_chapters: <int>
  questions_per_chapter: <int>
  chapters:
    - id: "<slug>"
      questions_generated: <int>
      questions_remaining: <int>
```

---

# 🔁 5. Algorithme de génération progressive

1. Lire `meta`, `plan`, `qcm`, `progress`
2. Déduire les options du run
3. Mettre à jour `meta.options_last_run`
4. Si QCM complet :
   - message explicatif
   - aucun ajout
   - status → complete
5. Sinon :
   - trouver le premier chapitre incomplet
   - générer jusqu’à `new_questions`
   - mettre à jour `progress`
   - marquer les chapitres terminés

---

# 🧠 6. Règles de génération des questions

- en français
- 4 réponses
- 1 seule correcte
- `correct` ∈ {0,1,2,3}
- explication factuelle et courte
- difficulté progressive dans un même chapitre

---

# ▶️ 7. Mode chat vs mode fichier

## 7.1 Mode fichier

- produire **uniquement** du YAML
- aucune explication textuelle
- modifications directement dans le fichier cible

## 7.2 Mode chat

L’assistant doit afficher **avant** le YAML :

Un message synthétique avec uniquement :

1. nombre de nouvelles questions
2. chapitre traité
3. numéro de question de reprise
4. indication si le QCM est complet

Si déjà complet :

> Le QCM est déjà complet. Aucune nouvelle question générée.

---

# 📦 8. Résumé des obligations

- interpréter le plan
- générer le QCM
- maintenir `progress`
- mémoriser options initiales et dernier run
- `new_questions`=10 par défaut
- ne jamais afficher les questions en mode chat
- produire un YAML unique, propre
- **écrire dans `dist/<output_file>`**
- **créer `dist/` si nécessaire**
- **output_file ≤ 20 caractères**

---

# ℹ️ Version

**Version : v12**  
**Nom du fichier : `xxxGenQCM.prompt.md`**
