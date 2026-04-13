# Prime — Chargement de contexte

Charge le contexte du second cerveau Joplin au debut de chaque session Claude.

## Prerequis

Lire `projet/joplin-config.json` pour obtenir `api_url` et `token`.
Toutes les requetes suivantes utilisent ces valeurs.

## Etapes

### 1. Lire CLAUDE.md

Lire le fichier `projet/CLAUDE.md` pour charger les regles absolues et la structure du systeme.

### 2. Trouver et lire la note index

Recuperer la liste de tous les tags :

```
GET {api_url}/tags?token={token}
```

Identifier le tag dont le titre est `index`. Recuperer son `id`.

Lister les notes associees a ce tag :

```
GET {api_url}/tags/{tag_id}/notes?token={token}&fields=id,title,body,updated_time
```

Lire le corps de la note index. C'est le panneau de direction du wiki — ne pas scanner d'autres notes.

Si le tag `index` n'existe pas ou qu'aucune note n'y est associee : signaler a l'utilisateur que le wiki n'est pas encore initialise et lui proposer de lancer `/ingest` pour commencer.

### 3. Trouver et lire la derniere daily note

Identifier le tag `daily`. Lister les notes associees :

```
GET {api_url}/tags/{tag_id}/notes?token={token}&fields=id,title,body,updated_time&order_by=updated_time&order_dir=DESC&limit=1
```

Lire le corps de la note daily la plus recente.

Si aucune daily n'existe : signaler a l'utilisateur qu'aucune session precedente n'a ete trouvee.

## Output attendu

Confirmer ce qui a ete charge en resumant :

- Nombre de categories dans l'index (compter les sections ou entrees de la note index)
- Date de la derniere daily note
- Points cles de la derniere session (actions, decisions, prochaines etapes extraits de la daily)

Exemple de sortie :

```
## Contexte charge

- CLAUDE.md : lu
- Index wiki : 3 categories, derniere mise a jour YYYY-MM-DD
- Derniere session : YYYY-MM-DD
  - Actions : ...
  - Prochaine etape : ...
```

## Regles

- Ne JAMAIS ecrire dans Joplin pendant /prime — c'est une operation de lecture uniquement
- Ne JAMAIS scanner toutes les notes du wiki — utiliser uniquement la note index comme point d'entree
- Si l'index n'existe pas ou est vide, le signaler clairement a l'utilisateur
