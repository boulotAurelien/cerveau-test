# Prime — Chargement de contexte

Charge le contexte du vault au debut de chaque session Claude Code.

## Etapes

1. **Lis `CLAUDE.md`** a la racine du vault — ce sont les instructions operationnelles
2. **Lis `wiki/index.md`** — c'est le panneau de direction du wiki
3. **Lis la derniere daily note** dans `wiki/Daily/` — c'est le resume de la session precedente

## Output attendu

Confirme ce qui a ete charge en resumant :

- Nombre de categories dans l'index
- Derniere daily note lue (date)
- Points cles de la derniere session

## Regles

- Ne JAMAIS ecrire dans le vault pendant /prime — c'est une operation de lecture uniquement
- Ne JAMAIS scanner tous les fichiers du wiki — utilise l'index comme point d'entree
- Si l'index n'existe pas ou est vide, le signaler a l'utilisateur
