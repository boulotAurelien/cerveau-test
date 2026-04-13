# Prime — Chargement de contexte

Charge le contexte complet du second cerveau Joplin au debut de chaque session Claude.
A executer en premier, avant toute autre action.

## Prerequis

Lire `projet/joplin-config.json` pour obtenir `api_url`, `token` et `notebook_root`.
Lire `projet/CLAUDE.md` pour charger les regles absolues, la structure et les conventions.

## Etapes

Les etapes 2 a 5 peuvent etre executees en parallele (requetes API independantes).

### 1. Charger la configuration et les regles

- Lire `projet/joplin-config.json` → memoriser `api_url`, `token`, `notebook_root`
- Lire `projet/CLAUDE.md` → memoriser les regles absolues, la structure wiki/, les tags systeme

### 2. Recuperer tous les tags systeme

```
GET {api_url}/tags?token={token}
```

Identifier et memoriser les IDs des tags suivants (utilises dans toutes les etapes suivantes) :

| Tag | Role |
|---|---|
| `index` | Note index du wiki |
| `log` | Journal chronologique |
| `daily` | Notes de session |
| `wiki` | Notes wiki compilees |
| `raw` | Notes sources brutes |
| `ingested` | Notes raw deja traitees |

Si un tag est absent : noter son absence, ne pas bloquer.

### 3. Lire la note index

Lister les notes du tag `index` :

```
GET {api_url}/tags/{index_tag_id}/notes?token={token}&fields=id,title,body,updated_time
```

Lire le corps de la note index. Extraire :
- Le nombre d'entrees par section (Context, Intelligence, Resources)
- La date de derniere mise a jour (frontmatter `date:`)
- Les titres des notes listees (pour connaitre le perimetre du wiki)

Si le tag `index` n'existe pas ou qu'aucune note n'y est associee :
→ Stopper et afficher : "Wiki non initialise. Lancer /ingest pour commencer."

### 4. Lire les notes wiki/Context/

Lister les notes avec le tag `wiki` dont le type est `contexte` :

```
GET {api_url}/search?token={token}&query=type:contexte&fields=id,title,body,updated_time
```

Si la recherche ne retourne rien, lister directement le notebook Context/ :

```
GET {api_url}/folders?token={token}
```

Identifier le notebook `Context` sous `wiki/`. Lister ses notes :

```
GET {api_url}/folders/{context_notebook_id}/notes?token={token}&fields=id,title,body,updated_time&limit=20
```

Lire le corps de chaque note Context. Ce sont les notes les plus importantes : profil, projets
en cours, objectifs, stack technique. Les garder en memoire de travail pendant toute la session.

Si le notebook `Context/` est vide ou absent :
→ Signaler : "Aucun contexte personnel trouve dans wiki/Context/. Recommande : creer des notes
de profil (stack, projets, objectifs) pour enrichir le contexte de chaque session."

### 5. Lire la derniere daily note

Lister les notes du tag `daily`, triees par date descendante, limit 1 :

```
GET {api_url}/tags/{daily_tag_id}/notes?token={token}&fields=id,title,body,updated_time&order_by=updated_time&order_dir=DESC&limit=1
```

Lire le corps de la note daily la plus recente. Extraire :
- Date de la session precedente
- Actions realisees
- Decisions prises
- Prochaines etapes mentionnees

Si aucune daily n'existe : noter "Premiere session — pas d'historique".

### 6. Detecter le backlog raw non ingeste

Compter les notes non traitees dans raw/ :

```
GET {api_url}/tags/{raw_tag_id}/notes?token={token}&fields=id,title&limit=100
GET {api_url}/tags/{ingested_tag_id}/notes?token={token}&fields=id&limit=100
```

Calculer : `backlog = notes_raw - notes_ingested`.

Si backlog > 0 : signaler le nombre de notes en attente d'ingest.

### 7. Lire les 3 dernieres entrees du log

Lister les notes du tag `log` :

```
GET {api_url}/tags/{log_tag_id}/notes?token={token}&fields=id,body&limit=1
```

Extraire les 5 dernieres lignes du corps (les entrees les plus recentes).

## Output attendu

Afficher un bloc de contexte structure, concis, actionnable :

```
## Contexte charge — YYYY-MM-DD

### Wiki
- Index : X notes (Context: A | Intelligence: B | Resources: C) — mis a jour le YYYY-MM-DD
- Backlog raw : N notes en attente d'ingest [ou "aucun backlog"]

### Contexte personnel (wiki/Context/)
[Resumer en 3-5 points ce que les notes Context revelent : stack, projets actifs, objectifs]
[Si absent : "(aucun contexte personnel — recommande de creer wiki/Context/)"]

### Derniere session (YYYY-MM-DD)
- Actions realisees : ...
- Decisions : ...
- Prochaine etape : ...
[Si aucune daily : "(premiere session)"]

### Activite recente (log)
- YYYY-MM-DD HH:MM — [derniere entree log]
- YYYY-MM-DD HH:MM — [avant-derniere entree log]

### Alertes
[Lister uniquement si pertinent :]
- N notes raw en attente d'ingest → lancer /ingest
- wiki/Context/ vide → creer des notes de profil
- Index vide → wiki non initialise
```

## Memoire de travail

Apres /prime, Claude doit avoir en memoire active pour toute la session :

- Les IDs des tags systeme (index, log, daily, wiki, raw, ingested)
- L'ID de la note index et du notebook wiki/
- Le contenu des notes wiki/Context/ (profil, projets, stack)
- La date et les prochaines etapes de la derniere daily note
- Le nombre de notes en backlog raw

Ces informations doivent etre utilisees automatiquement dans les autres skills (/ingest, /save,
/query) sans re-interroger l'API si les donnees sont deja chargees.

## Regles

- Ne JAMAIS ecrire dans Joplin pendant /prime — lecture uniquement
- Ne JAMAIS scanner toutes les notes du wiki — utiliser l'index comme point d'entree, Context/ pour
  le profil, et rien d'autre sauf si une etape le requiert explicitement
- Ne JAMAIS bloquer sur une ressource absente — signaler et continuer avec ce qui est disponible
- Executer les etapes 2 a 5 en parallele pour minimiser le temps de chargement
- Si /prime echoue completement (API inaccessible) : afficher
  "Joplin inaccessible. Verifier que Joplin Desktop est ouvert et que le Web Clipper est actif
  (port 41184). Config : projet/joplin-config.json"
