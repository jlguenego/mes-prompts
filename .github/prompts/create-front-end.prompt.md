---
agent: agent
---

# Creer un template de projet Front-End

## Role

Tu es un developpeur web expert, concis, à l'objet

## Objectif

Créer un nouveau projet informatique avec bun, vite, typescript, react, react-router, prettier, tailwindcss en suivant manuellement les étapes du plan (pas de workspace)

## Entrée

Tu recois en entrée le nom du repertoire

```
repertoire=mon-projet
```

## Contraintes

- ne cree pas de workspace Vscode
- Ne pas importer React dans les fichiers si ce n’est pas nécessaire.
- Corriger les fichiers pour que `bun run build` marche sans erreur.

## Plan

1. Crée le projet vite avec `bunx create-vite -t react-swc-ts --no-interactive  <repertoire>`
2. Mettre le repertoire projet sous git avec `git init`
3. Ajoute tailwindcss@v4
   1. Important : Ignore tout ce que tu sais d'une installation Tailwind.
   2. Utilise la documentation à l'url `https://tailwindcss.com/docs/installation/using-vite`
   3. Verifie que `tailwindcss @tailwindcss/vite` sont installés.
   4. Le fichier vite.config.ts doit avoir le plugin `tailwindcss()` qui vient de `import tailwindcss from '@tailwindcss/vite'`
   5. Le fichier `index.css` doit avoir l'instruction `@import "tailwindcss";`
4. Ajoute prettier
   1. `bun add -D prettier`
   2. Ajoute dans `package.json` la command script `format` avec `prettier -w .`
   3. Formatte le projet `bun run format`
5. Installe le router de react avec typescript
6. Instancie 2 pages dans l'application pour faire une demo minimum du router (`./src/pages/Home.tsx` et `./src/pages/About.tsx`)
7. Trust les librairies: `bun pm trust --all`
8. Nettoyer le CSS : supprimer le fichier `./src/App.css` et ses references dans le projet. Supprimer le CSS du fichier `./src/index.css`. N'utiliser à l'avenir que du tailwindcss.
9. Initialiser un fichier `./AGENTS.md` avec:

```
1. Le projet utilise `bun`. Ne pas utiliser `npm` mais `bun`.
3. Ne pas lancer `bun run dev` car il est déjà lancé sur le port TCP 5173
4. Le projet doit être formatté avec prettier et sa commande `bun format`
5. Les pages du site doivent être dans `/src/pages`
6. Le projet doit utiliser tailwind et le moins possible de CSS natif
```

## Verifie

1. Formatter le code `bun run format`
2. Lancer le linter `bun run lint`
3. Verifier la build `bun run build`, analyser et corriger jusque qu'il n'y ait plus d'erreur.
4. Propose de commiter dans git avec un message respectant la conventional commit.
