# Lint — Health-check du wiki

Verifie l'integrite et la coherence du wiki Joplin. A lancer periodiquement (1x/semaine recommande).

## Prerequis

Lire `projet/joplin-config.json` pour obtenir `api_url` et `token`.

## Checks

### 1. Recuperer toutes les notes wiki

Trouver le tag `wiki` :

```
GET {api_url}/tags?token={token}
```

Lister toutes les notes avec ce tag :

```
GET {api_url}/tags/{wiki_tag_id}/notes?token={token}&fields=id,title,body,updated_time
```

Lire le corps de chaque note pour les checks suivants.

### 2. Notes orphelines

Pour chaque note wiki, verifier qu'elle possede au moins un lien sortant ou entrant.

- **Lien sortant** : le corps de la note contient au moins un lien `[Titre](:/id)`
- **Lien entrant** : une autre note ou la note index contient un lien `[...](:/cette_note_id)`

Lister les notes sans aucun lien entrant ni sortant : ce sont les orphelines.

### 3. Liens casses

Scanner tous les liens `[Titre](:/id)` trouves dans les corps de toutes les notes wiki.

Pour chaque `id` reference, verifier que la note existe :

```
GET {api_url}/notes/{id}?token={token}&fields=id,title
```

Si la reponse est une erreur 404 ou vide : le lien est casse. Lister : note source + titre du lien + id introuvable.

### 4. Index a jour

Lire la note avec le tag `index`.

Comparer la liste des IDs de toutes les notes wiki avec les liens `(:/id)` presents dans la note index.

Lister les notes wiki absentes de l'index (presentes dans le wiki mais non referencees dans l'index).

Compter le nombre de lignes de la note index. Si > 200 lignes, recommander la creation de sous-index par categorie.

### 5. Coherence du frontmatter

Pour chaque note wiki, verifier que le corps commence par un frontmatter YAML valide contenant les champs obligatoires :
- `date` (format YYYY-MM-DD)
- `type` (valeur parmi : note, contexte, recherche, ressource, daily)
- `status` (valeur parmi : active, archive)

Lister les notes avec frontmatter absent ou incomplet.

### 6. Ecrire dans le log

Trouver la note avec le tag `log`. Ajouter en fin de corps :

```
PUT {api_url}/notes/{log_note_id}?token={token}
```

Entree a ajouter :

```
YYYY-MM-DD HH:MM — Lint : X orphelines, X liens casses, X notes absentes de l'index, X frontmatter invalides
```

## Rapport

Afficher :

```
## Lint termine

### Orphelines : X
- [Titre](:/id) — aucun lien entrant ni sortant
- ...

### Liens casses : X
- Dans [Note source](:/id) : lien [Titre](:/id_introuvable) → note cible inexistante
- ...

### Index incomplet : X notes manquantes
- [Titre](:/id) — absente de l'index
- ...

### Index : XX lignes (OK | ATTENTION > 200 — creer des sous-index)

### Frontmatter invalide : X
- [Titre](:/id) — champs manquants : date, type, status
- ...

### Actions recommandees
- [ ] Ajouter des liens vers/depuis les notes orphelines
- [ ] Corriger ou supprimer les liens casses
- [ ] Ajouter les notes manquantes dans l'index
- [ ] Corriger le frontmatter des notes invalides
```

## Regles

- Ne JAMAIS corriger automatiquement — proposer les corrections, l'utilisateur valide
- Ne JAMAIS modifier les notes dans raw/
- Ne JAMAIS supprimer de notes — proposer l'archivage (`status: archive` dans le frontmatter)
- Si le tag `wiki` n'existe pas, signaler que le wiki n'est pas encore initialise
