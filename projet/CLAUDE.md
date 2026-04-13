# Second Cerveau — Joplin + Claude

## Configuration

La configuration de l'API Joplin se trouve dans `projet/joplin-config.json` :
- `api_url` : URL de base de l'API REST Joplin (par defaut `http://localhost:41184`)
- `token` : token d'authentification Joplin (Menu Joplin → Outils → Options → Web Clipper)
- `notebook_root` : nom du notebook racine dans Joplin (par defaut `Second Cerveau`)

Toutes les requetes API doivent inclure `?token=<TOKEN>` en query parameter.

## Regles absolues

1. Ne JAMAIS modifier une note dans le notebook `raw/` — c'est l'espace humain, immutable
2. Ne JAMAIS creer de note orpheline — chaque note wiki doit avoir au moins un lien entrant ou sortant
3. Ne JAMAIS ecrire dans Joplin sans passer par un skill (/ingest, /save, /query)
4. Ne JAMAIS supprimer une note wiki — archiver en changeant `status: archive` dans le frontmatter
5. Ne JAMAIS inventer d'information absente du wiki — signaler quand la donnee manque

## Architecture 3 couches (Karpathy LLM Wiki)

| Couche           | Notebook Joplin          | Proprietaire | Regle                                              |
| ---------------- | ------------------------ | ------------ | -------------------------------------------------- |
| Layer 1 — Raw    | `Second Cerveau/raw/`    | Humain       | Immutable. Inputs bruts (clippings, docs, notes).  |
| Layer 2 — Wiki   | `Second Cerveau/wiki/`   | LLM          | Compile via `/ingest`. Concepts, resumes, index.   |
| Layer 3 — Schema | `projet/CLAUDE.md`       | Humain       | Structure et conventions. Definit raw → wiki.      |

## Fonctionnement

L'humain browse les notes dans Joplin. Le LLM ecrit et maintient le wiki via l'API REST. Le wiki compile et grandit.

- **raw/** contient le bordel humain : articles clippes, docs, notes brutes. Le LLM lit mais ne touche JAMAIS.
- **wiki/** contient la connaissance compilee : notes structurees, index, log. Le LLM est responsable de la qualite.
- La note avec le tag `#index` dans wiki/ est le panneau de direction. Le LLM la lit EN PREMIER. Economise les tokens.
- La note avec le tag `#log` dans wiki/ est le journal chronologique append-only.
- Les notes raw traitees sont taguees `#ingested` pour ne pas les retraiter.

## Operations disponibles

| Commande  | Role                                       | Quand l'utiliser                            |
| --------- | ------------------------------------------ | ------------------------------------------- |
| `/prime`  | Charger le contexte au debut d'une session | A chaque nouvelle session Claude            |
| `/ingest` | Compiler raw/ → wiki/                      | Apres avoir ajoute du contenu dans raw/     |
| `/save`   | Sauvegarder l'etat de la session           | En fin de session de travail                |
| `/query`  | Recherche profonde dans le wiki            | Pour trouver de l'information dans le wiki  |
| `/lint`   | Health-check du wiki                       | Periodiquement (1x/semaine recommande)      |

## API Joplin — Endpoints cles

Lire la config dans `projet/joplin-config.json` pour obtenir `api_url` et `token`.

```
# Lister les notebooks
GET {api_url}/notebooks?token={token}

# Lister les notes d'un notebook
GET {api_url}/folders/{notebook_id}/notes?token={token}&fields=id,title,body,updated_time

# Lire une note
GET {api_url}/notes/{note_id}?token={token}&fields=id,title,body,tags,updated_time

# Creer une note
POST {api_url}/notes?token={token}
Body JSON : { "title": "...", "body": "...", "parent_id": "{notebook_id}" }

# Modifier une note
PUT {api_url}/notes/{note_id}?token={token}
Body JSON : { "title": "...", "body": "..." }

# Lister les tags
GET {api_url}/tags?token={token}

# Creer un tag
POST {api_url}/tags?token={token}
Body JSON : { "title": "nom-du-tag" }

# Ajouter un tag a une note
POST {api_url}/tags/{tag_id}/notes?token={token}
Body JSON : { "id": "{note_id}" }

# Notes par tag
GET {api_url}/tags/{tag_id}/notes?token={token}&fields=id,title,body,updated_time

# Recherche full-text
GET {api_url}/search?token={token}&query={terme}&fields=id,title,body
```

## Conventions Joplin

- **Liens internes** : `[Titre de la note](:/note_id)` — utiliser l'ID Joplin de la note cible
- **Tags** : utiliser les tags Joplin natifs pour la navigation et le filtrage
- **Frontmatter YAML obligatoire** dans le corps de chaque note wiki (premiere section de la note) :

```yaml
---
date: YYYY-MM-DD
tags: []
type: note | contexte | recherche | ressource | daily
status: active | archive
---
```

## Tags systeme

| Tag          | Role                                                  |
| ------------ | ----------------------------------------------------- |
| `#index`     | Note index du wiki (panneau de direction)             |
| `#log`       | Note log des operations (journal chronologique)       |
| `#daily`     | Notes daily generees par /save                        |
| `#ingested`  | Note raw deja traitee par /ingest (ne pas retraiter)  |
| `#wiki`      | Toute note wiki compilee                              |
| `#raw`       | Note source brute humaine                             |

## Structure wiki/

| Sous-notebook        | Contenu                                         |
| -------------------- | ----------------------------------------------- |
| `wiki/Context/`      | Notes de contexte : profil, objectifs, projets  |
| `wiki/Intelligence/` | Decisions, recherches, analyses, benchmarks     |
| `wiki/Resources/`    | Templates, patterns, snippets reutilisables     |
| `wiki/Daily/`        | Journal quotidien auto-genere par /save         |

Ces sous-notebooks se creent au besoin via l'API. Ne JAMAIS creer un notebook vide.
