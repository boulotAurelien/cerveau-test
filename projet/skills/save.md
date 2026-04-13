# Save — Sauvegarde de session

Sauvegarde l'etat de la session en cours dans Joplin. A lancer en fin de session.

## Prerequis

Lire `projet/joplin-config.json` pour obtenir `api_url`, `token` et `notebook_root`.

## Etapes

### 1. Determiner la date du jour

La date du jour au format `YYYY-MM-DD` sera le titre de la daily note.

### 2. Chercher si une daily note existe deja pour aujourd'hui

Trouver le tag `daily` :

```
GET {api_url}/tags?token={token}
```

Lister les notes avec ce tag :

```
GET {api_url}/tags/{daily_tag_id}/notes?token={token}&fields=id,title,body,updated_time
```

Chercher une note dont le titre est exactement `YYYY-MM-DD` (date du jour).

### 3a. Si la daily note n'existe pas — la creer

Trouver le notebook `wiki/Daily/` :

```
GET {api_url}/notebooks?token={token}
```

Identifier le notebook `Daily` sous `wiki/`. Creer la note :

```
POST {api_url}/notes?token={token}
Body JSON : {
  "title": "YYYY-MM-DD",
  "body": "...",
  "parent_id": "{daily_notebook_id}"
}
```

Corps de la note :

```markdown
---
date: YYYY-MM-DD
tags: [daily]
type: daily
status: active
---

## Actions

- ...

## Decisions

- ...

## Prochaine etape

- ...
```

Ajouter le tag `daily` a la note creee :

```
POST {api_url}/tags/{daily_tag_id}/notes?token={token}
Body JSON : { "id": "{note_id}" }
```

### 3b. Si la daily note existe deja — ajouter le contenu

Lire le corps existant de la note. Ajouter le nouveau contenu a la suite (ne jamais ecraser) :

```
PUT {api_url}/notes/{note_id}?token={token}
Body JSON : { "body": "...contenu existant...\n\n---\n\n## Session HH:MM\n\n### Actions\n- ...\n\n### Decisions\n- ...\n\n### Prochaine etape\n- ..." }
```

### 4. Mettre a jour la note index

Si de nouvelles notes wiki ont ete creees pendant la session, les ajouter dans la note index.

Trouver la note avec le tag `index`, lire son contenu, verifier les entrees manquantes, mettre a jour :

```
PUT {api_url}/notes/{index_note_id}?token={token}
Body JSON : { "body": "...index mis a jour..." }
```

### 5. Ecrire dans le log

Trouver la note avec le tag `log`. Ajouter en fin de corps :

```
PUT {api_url}/notes/{log_note_id}?token={token}
```

Entree a ajouter :

```
YYYY-MM-DD HH:MM — Save : daily note creee/mise a jour, index verifie
```

### 6. Confirmation

Afficher :

```
## Save termine

- Daily note : YYYY-MM-DD (creee | mise a jour)
- Index : mis a jour (X entrees ajoutees) | aucun changement
- Log : mis a jour
```

## Regles

- Ne JAMAIS supprimer ou ecraser du contenu existant dans une daily note — uniquement ajouter
- Ne JAMAIS modifier les notes dans raw/
- Ne JAMAIS creer de note orpheline
- Executer directement sans demander confirmation a l'utilisateur
- Si le notebook `wiki/Daily/` n'existe pas, le creer via l'API avant de creer la note
