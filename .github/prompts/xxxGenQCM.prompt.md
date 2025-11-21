# 🧠 Commande `xxxGenQCM`

## Spécification complète – Assistant IA intégré à l’éditeur

### (Compatible Copilot, Cursor, Windsurf, Codeium, et tout moteur de complétion IA)

---

# 📘 1. Objectif général

Cette spécification décrit comment un **assistant IA intégré à l’éditeur** doit exécuter la commande :

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
7. **Fournir en mode chat un message explicatif synthétique**
8. Générer un fichier YAML autosuffisant (`meta + plan + qcm + progress`)

---

# 📝 2. Analyse du fichier Markdown d’entrée

L’assistant doit comprendre des plans de formation variés, même mal formatés.

---

## 2.1. Détection du titre de la formation

Le titre est extrait automatiquement, même si :

- ce n’est pas un `#`,
- il est dans une forme libre,
- il apparaît sous forme de section textuelle.

Le titre sert à :

- générer `qcm_title` si absent :  
  **“QCM sur <Titre>”**
- générer le nom de fichier par défaut :  
  **`qcm-<slug-titre>.yaml`**

---

## 2.2. Détection du programme

Le programme peut être signalé par :

- Programme, Contenu, Modules, Curriculum, Plan, etc.

L’assistant doit reconnaître :

- Chapitres (`##`, `Module X`, `Partie X`, `1.`, etc.)
- Notions (listes, phrases, paragraphes, concepts séparés par virgules…)

Il doit reconstruire une structure propre :

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

Ignorer :

- prérequis
- objectifs
- public
- matériel requis
- logistique
- introduction non pédagogique

---

# ⚙️ 3. Paramètres de la commande `xxxGenQCM`

La commande peut être écrite :

- dans un commentaire YAML d’un fichier
- ou dans un message chat/agent

Exemples :

```yaml
# xxxGenQCM
# xxxGenQCM questions_per_chapter=40 difficulty=facile
# xxxGenQCM new_questions=12
```

---

## 3.1. Paramètres disponibles

| Paramètre               | Exemple                         | Défaut            | Description                                   |
| ----------------------- | ------------------------------- | ----------------- | --------------------------------------------- |
| `questions_per_chapter` | `questions_per_chapter=40`      | **20**            | Nbre de questions par chapitre                |
| `language`              | `language=en`                   | **fr**            | QCM **toujours en français**                  |
| `difficulty`            | `difficulty=moyen`              | **progressive**   | Difficulté                                    |
| `qcm_title`             | `qcm_title="QCM Docker"`        | “QCM sur <Titre>” | Titre du QCM                                  |
| `output_file`           | `output_file="qcm-docker.yaml"` | `qcm-<slug>.yaml` | Nom du fichier                                |
| `new_questions`         | `new_questions=15`              | **10**            | Maximum de nouvelles questions lors de ce run |

### 3.2. Priorité des paramètres

1. Paramètres dans la commande
2. Paramètres dans un éventuel bloc HTML
3. Valeurs par défaut

---

# 🧾 4. Structure du fichier YAML généré

Le fichier contient obligatoirement **4 sections** :

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

### 4.1.1. `meta.options_original`

- Snapshot **immuable** de la toute première exécution.
- Contient :
  - language
  - questions_per_chapter
  - difficulty
  - qcm_title
  - output_file
  - new_questions (valeur d’origine ou défaut = 10)

### 4.1.2. `meta.options_last_run`

- Options **réelles** utilisées lors de la dernière exécution.
- Doit être mis à jour à chaque run.
- Contient aussi `new_questions`.

---

## 4.2. Section `plan`

Représente le plan de formation interprété.

Structure :

```yaml
plan:
  chapters:
    - id: "<slug>"
      title: "<Titre>"
      notions:
        - "<notion>"
```

Ce plan devient la **source de vérité** des générations suivantes.

---

## 4.3. Section `qcm`

Structure :

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

L’assistant **ajoute** des questions.  
Il ne doit **jamais** modifier ou supprimer les questions existantes.

---

## 4.4. Section `progress`

Permet à l’assistant de savoir où reprendre :

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

Lors de chaque appel :

1. Lire `meta`, `plan`, `qcm`, `progress`.
2. Déterminer les options du run :  
   commande → HTML → défauts.
3. Mettre à jour `meta.options_last_run`.
4. **Si le QCM est complet** :
   - ne générer aucune question
   - passer en message chat explicatif (voir section 7.2)
   - `progress.status = complete`
5. Sinon :
   - identifier le premier chapitre incomplet
   - générer **au maximum `new_questions` questions** ou moins si le chapitre atteint son quota
   - mettre à jour `progress`
   - mettre à jour `completed_chapters` le cas échéant
   - si tous complets → `status = complete`

---

# 🧠 6. Règles de génération des questions

- Toujours en **français**
- 4 réponses obligatoires
- 1 seule correcte
- `correct` ∈ {0,1,2,3}
- explication courte, factuelle
- difficulté :
  - progressive → simple → intermédiaire → avancée
  - selon l’ordre des questions du chapitre

---

# ▶️ 7. Comportement selon le mode d’exécution

---

## 7.1. Mode **complétion dans fichier**

- L’assistant **ne doit écrire que du YAML**
- Jamais de texte en dehors du YAML
- Les mises à jour se font dans le fichier actuel uniquement

---

## 7.2. Mode **agent / chat** (Cursor, Windsurf…)

Avant de produire le YAML :

### ➤ L’assistant doit obligatoirement afficher un **message explicatif synthétique**, contenant UNIQUEMENT :

1. **Combien de nouvelles questions** ont été générées lors du run
2. **Dans quel chapitre** la génération a repris
3. **À partir de quel numéro de question** la génération a repris
4. **Si le QCM est maintenant complet ou non**

→ **Aucune question ni réponse ne doit être visible dans la synthèse.**  
→ Aucun extrait du YAML ne doit être montré dans le message explicatif.

Ensuite seulement, produire :

```yaml
# FILE: <output_file>
# xxxGenQCM ...
meta: ...
plan: ...
qcm: ...
progress: ...
```

Si le QCM est déjà complet :

> « Le QCM est déjà complet. Aucune nouvelle question générée. »

---

# 📦 8. Résumé des obligations

- Extraire & encoder le plan → `plan`
- Générer le QCM → `qcm`
- Suivre l’avancement → `progress`
- Mémoriser :
  - options initiales (`meta.options_original`)
  - options du dernier run (`meta.options_last_run`)
- `new_questions` = 10 par défaut
- En mode chat → message explicatif synthétique
- Ne jamais afficher ou prévisualiser les questions générées dans ce message
- YAML final = unique, auto-suffisant, non destructif

---

# ℹ️ Version

**Version : v10**  
**Nom du fichier : `xxxGenQCM.prompt.md`**
