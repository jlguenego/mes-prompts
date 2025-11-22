# 🧠 Commande `xxxGenQCM`

## Spécification complète – Assistant IA intégré à l’éditeur

### (Compatible Copilot, Cursor, Windsurf, Codeium, et tout moteur de complétion IA)

---

# 📘 1. Objectif général

Cette spécification décrit comment un **assistant IA intégré à l’éditeur** doit exécuter la commande :

```text
xxxGenQCM [param=value] [param=value] …
```

Sa mission : Analyser un plan Markdown, générer un QCM YAML progressif, mémoriser l'état, limiter les questions par run, et écrire dans `dist/` avec nom ≤20 caractères.

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

Le titre est extrait automatiquement, même mal formaté ou non titré. Sert à générer `qcm_title` (défaut : “QCM sur <Titre>”) et le nom de fichier `qcm-<slug>.yaml` (≤20 caractères).

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
  plan_chapters:
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

La commande est lancée direcement en mode chat.

Exemples :

```
xxxGenQCM
xxxGenQCM questions_per_chapter=40 difficulty=facile
xxxGenQCM new_questions=12
```

---

## 3.1. Paramètres disponibles

| Paramètre               | Exemple                         | Défaut            | Description                                                      |
| ----------------------- | ------------------------------- | ----------------- | ---------------------------------------------------------------- |
| `questions_per_chapter` | `questions_per_chapter=40`      | **20**            | Nombre de questions par chapitre                                 |
| `language`              | `language=en`                   | **fr**            | QCM produit en français uniquement                               |
| `difficulty`            | `difficulty=moyen`              | **progressive**   | Difficulté des questions : facile, moyen, difficile, progressive |
| `qcm_title`             | `qcm_title="QCM Docker"`        | “QCM sur <Titre>” | Titre du QCM                                                     |
| `output_file`           | `output_file="qcm-docker.yaml"` | `qcm-<slug>.yaml` | Nom du fichier (≤ 20 caractères)                                 |
| `new_questions`         | `new_questions=15`              | **10**            | Maximum de nouvelles questions générées lors du run              |

---

## 3.2. Règles prioritaires

1. Paramètres directement fournis dans la commande
2. Valeurs par défaut

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

#### Algorithme de calcul du slug

1. **Normalisation du titre** : Prendre le titre de la formation, le convertir en minuscules.
2. **Suppression des accents** : Remplacer les caractères accentués par leurs équivalents non accentués (ex. : é → e, à → a).
3. **Remplacement des espaces** : Remplacer tous les espaces par des tirets (-).
4. **Suppression des mots vides** : Retirer les mots courants comme "de", "la", "le", "les", "et", "a", "un", "une", "des", "du", "au", "aux", "sur", "pour", "avec", etc.
5. **Nettoyage des caractères spéciaux** : Supprimer ou remplacer les caractères non alphanumériques (sauf tirets) par des tirets ou rien.
6. **Troncature si nécessaire** : Si la longueur de `qcm-<slug>.yaml` dépasse 20 caractères, tronquer le slug en gardant des mots entiers autant que possible, en priorisant les premiers mots significatifs.
7. **Vérification finale** : Assurer que le slug ne commence ni ne finit par un tiret, et qu'il n'y a pas de tirets consécutifs.

Exemple : Pour le titre "Formation sur Docker et Kubernetes", le slug devient "formation-docker-kubernetes", donnant "qcm-formation-docker-kubernetes.yaml" (29 caractères → tronquer à "qcm-docker-kubernetes.yaml" si nécessaire).

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

Le fichier contient les sections suivantes :

```yaml
title: "<Titre du QCM>"
chapters:
  - id: "<slug>"
    title: "<Titre>"
    questions:
      - id: "<unique>"
        question: "<texte>"
        answers: ["A", "B", "C", "D"]
        correct: <0-3>
        explanation: "<explication>"
meta:
plan:
progress:
```

---

## 4.1. Section `title`

Contient le titre du QCM :

- `title`: “QCM sur <Titre>” ou valeur personnalisée

---

## 4.2. Section `chapters`

Structure complète des chapitres avec questions :

```yaml
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

## 4.3. Section `meta`

Contient : `formation_title`, `qcm_title`, `output_file`, `language`, `difficulty`, `questions_per_chapter`.

### 4.3.1 `meta.options_original`

Snapshot immuable du premier run (language, questions_per_chapter, difficulty, qcm_title, output_file, new_questions).

### 4.3.2 `meta.options_last_run`

Mis à jour à chaque run (idem + new_questions).

---

# 4.4. Section `plan`

Structure exacte :

```yaml
plan:
  plan_chapters:
    - id: "<slug>"
      title: "<Titre>"
      notions:
        - "<notion>"
```

---

# 4.5. Section `progress`

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

### Qualité pédagogique des questions

Les questions doivent être pertinentes, intelligentes et utiles pédagogiquement.

Pour chaque question :

- elle doit cibler une notion précise du plan
- elle doit évaluer une compréhension réelle, pas une définition triviale
- elle doit mélanger : compréhension, application, analyse
- elle doit être réaliste, issue de cas rencontrés dans la pratique
- elle doit éviter les pièges gratuits
- elle doit éviter le hors-sujet
- elle doit apporter une explication après la réponse correcte

---

# ▶️ 7. Mode chat

L’assistant doit afficher **avant** le YAML :

Un message synthétique avec uniquement :

1. nombre de nouvelles questions
2. chapitre traité
3. numéro de question de reprise
4. indication si le QCM est complet

Le fichier YAML est automatiquement écrit dans dist/<output_file> sans afficher son contenu dans le chat.

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
- produire un YAML unique, propre avec `title`, `chapters`, `meta`, `plan`, `progress`
- **écrire dans `dist/<output_file>`**
- **créer `dist/` si nécessaire**
- **output_file ≤ 20 caractères**

---

# ℹ️ Version

**Version : v12**  
**Nom du fichier : `xxxGenQCM.prompt.md`**
