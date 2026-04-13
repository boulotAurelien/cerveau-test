# Ingest — Compilation raw/ vers wiki/

Compile les notes brutes du notebook `raw/` en notes wiki structurees.

## Prerequis

Lire `projet/joplin-config.json` pour obtenir `api_url`, `token` et `notebook_root`.

## Etapes

### 1. Trouver le notebook raw/

Lister tous les notebooks :

```
GET {api_url}/notebooks?token={token}
```

Identifier le notebook dont le chemin correspond a `{notebook_root}/raw/` (ou ses sous-notebooks : `clippings`, `docs`, `notes`). Recuperer leurs `id`.

### 2. Lister les notes raw non traitees

Pour chaque sous-notebook raw (`clippings`, `docs`, `notes`) :

```
GET {api_url}/folders/{notebook_id}/notes?token={token}&fields=id,title,body,updated_time
```

Pour chaque note recuperee, verifier si elle possede le tag `ingested` :

```
GET {api_url}/notes/{note_id}/tags?token={token}
```

- Si le tag `ingested` est present → skip (deja traitee)
- Si absent → a traiter

### 3. Traiter chaque note non traitee

Pour chaque note a traiter :

1. **Lire** le contenu complet (title + body)
2. **Extraire** les concepts cles, faits, decisions, insights importants
3. **Decider** : creer une nouvelle note wiki OU enrichir une note existante
   - Chercher dans le wiki une note sur le meme sujet via :
     ```
     GET {api_url}/search?token={token}&query={mot_cle}&fields=id,title,body
     ```
   - Sujet existant → enrichir la note wiki correspondante
   - Sujet nouveau → creer dans le bon sous-notebook wiki/

### 4. Creer ou enrichir la note wiki

**Identifier le bon sous-notebook wiki/** selon le type de contenu :
- Profil, objectifs, projets → `wiki/Context/`
- Decisions, recherches, analyses → `wiki/Intelligence/`
- Templates, patterns, snippets → `wiki/Resources/`

**Creer une nouvelle note wiki** :

```
POST {api_url}/notes?token={token}
Body JSON : {
  "title": "Titre de la note",
  "body": "...",
  "parent_id": "{wiki_subnotebook_id}"
}
```

Le corps de la note doit commencer par le frontmatter YAML, suivi du contenu structure :

```markdown
---
date: YYYY-MM-DD
tags: []
type: recherche | contexte | ressource | note
status: active
source_note_id: {id_de_la_note_raw}
---

## Resume

2-3 phrases capturant l'essentiel.

## Concepts cles

- Concept 1
- Concept 2
- ...

## Details

Sections structurees si le contenu est riche.

## Liens

- [Note connexe 1](:/note_id_1)
- [Note connexe 2](:/note_id_2)
```

**Enrichir une note wiki existante** :

```
PUT {api_url}/notes/{note_id}?token={token}
Body JSON : { "body": "...contenu enrichi..." }
```

Ajouter le nouveau contenu sans supprimer l'existant. Mettre a jour la date dans le frontmatter.

**Ajouter les tags wiki** a la note creee/enrichie :

```
POST {api_url}/tags/{wiki_tag_id}/notes?token={token}
Body JSON : { "id": "{note_wiki_id}" }
```

### 5. Marquer la note raw comme traitee

Trouver ou creer le tag `ingested` :

```
GET {api_url}/tags?token={token}
```

Si le tag `ingested` n'existe pas :

```
POST {api_url}/tags?token={token}
Body JSON : { "title": "ingested" }
```

Ajouter le tag a la note raw :

```
POST {api_url}/tags/{ingested_tag_id}/notes?token={token}
Body JSON : { "id": "{note_raw_id}" }
```

### 6. Mettre a jour la note index

Trouver la note avec le tag `index`. La lire, puis ajouter les nouvelles entrees dans la section appropriee :

```
PUT {api_url}/notes/{index_note_id}?token={token}
Body JSON : { "body": "...index mis a jour..." }
```

Format d'entree dans l'index :

```
| [Titre de la note](:/note_id) | type | YYYY-MM-DD | Resume en une phrase |
```

### 7. Ecrire dans le log

Trouver la note avec le tag `log`. Ajouter en fin de corps :

```
PUT {api_url}/notes/{log_note_id}?token={token}
```

Entree a ajouter :

```
YYYY-MM-DD HH:MM — Ingest : X notes scannees, Y nouvelles, Z enrichies
```

### 8. Rapport final

Afficher :

```
## Ingest termine

- Notes scannees : X
- Nouvelles notes wiki : Y (liste des titres)
- Notes enrichies : Z (liste des titres)
- Skipped (deja traitees) : W
- Index mis a jour : oui/non
- Log mis a jour : oui/non
```

## Regles

- Ne JAMAIS modifier, renommer ou supprimer une note dans raw/ — immutable
- Ne JAMAIS creer de note superficielle — si une note n'apporte rien de nouveau, ne pas creer de note wiki
- Privilegier l'enrichissement d'une note existante plutot que la creation d'une nouvelle
- Les notes wiki sont des syntheses, pas des copies — reformuler, structurer, extraire la valeur
- Ne JAMAIS creer de note orpheline — ajouter au moins un lien vers une note connexe ou depuis l'index
- Utiliser les liens natifs Joplin `[Titre](:/id)` pour tous les liens internes
