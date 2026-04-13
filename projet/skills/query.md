# Query — Recherche profonde dans le wiki

Recherche et synthetise de l'information a partir du wiki Joplin.

## Prerequis

Lire `projet/joplin-config.json` pour obtenir `api_url` et `token`.

## Etapes

### 1. Lire la note index

Trouver le tag `index` :

```
GET {api_url}/tags?token={token}
```

Lister les notes associees :

```
GET {api_url}/tags/{tag_id}/notes?token={token}&fields=id,title,body,updated_time
```

Lire le corps de la note index. Identifier quelle(s) categorie(s) et quelle(s) note(s) sont les plus susceptibles de contenir la reponse a la question posee.

Ne pas scanner d'autres notes a ce stade.

### 2. Naviguer vers les notes pertinentes

Pour chaque note identifiee dans l'index, recuperer son contenu complet :

```
GET {api_url}/notes/{note_id}?token={token}&fields=id,title,body,updated_time
```

Si une note contient des liens `[Titre](:/id)` vers d'autres notes pertinentes, les lire egalement.

Si la note index ne permet pas d'identifier de notes pertinentes, utiliser la recherche full-text :

```
GET {api_url}/search?token={token}&query={mots_cles_de_la_question}&fields=id,title,body
```

### 3. Synthetiser

Formuler une reponse structuree basee uniquement sur le contenu des notes wiki lues.

- Citer systematiquement la source : `[Titre de la note](:/note_id)`
- Si plusieurs notes contribuent a la reponse, les citer toutes
- Ne jamais inventer ou extrapoler au-dela du contenu des notes

Si la donnee n'existe pas dans le wiki :

```
L'information demandee n'est pas presente dans le wiki.
Sources consultees : [liste des notes lues]
Suggestion : ajouter cette information via /ingest ou manuellement dans raw/.
```

### 4. Proposer un enrichissement (optionnel)

Si la recherche produit une nouvelle synthese utile (croisement de plusieurs notes, conclusion emergente non presente dans une note existante) :

- Proposer a l'utilisateur de creer ou enrichir une note wiki avec cette synthese
- Montrer un apercu du contenu propose
- Ne JAMAIS ecrire dans Joplin sans validation explicite de l'utilisateur

Si l'utilisateur valide, creer la note via :

```
POST {api_url}/notes?token={token}
Body JSON : { "title": "...", "body": "...", "parent_id": "{notebook_id}" }
```

### 5. Ecrire dans le log

Trouver la note avec le tag `log`. Ajouter en fin de corps :

```
PUT {api_url}/notes/{log_note_id}?token={token}
```

Entree a ajouter :

```
YYYY-MM-DD HH:MM — Query : "{question posee}" → [Note source](:/note_id)
```

## Regles

- Ne JAMAIS inventer d'information absente du wiki — si la donnee n'existe pas, le dire clairement
- Ne JAMAIS scanner toutes les notes du wiki — utiliser l'index comme point d'entree, recourir a la recherche full-text en second recours
- Ne JAMAIS modifier les notes dans raw/
- Toujours citer la ou les notes sources dans la reponse avec leur lien `[Titre](:/id)`
- Ne JAMAIS ecrire dans le wiki sans validation explicite de l'utilisateur
