# Second Brain Kit — Obsidian + Claude Code

Un vault Obsidian preconfigure pour donner une memoire permanente a Claude Code. Base sur l'architecture 3 couches d'Andrej Karpathy et les patterns internes de Claude Code.

## Ce que contient ce kit

- Un vault Obsidian pret a l'emploi avec la structure 3 couches (raw/wiki/schema)
- 5 slash commands pour Claude Code : `/prime`, `/ingest`, `/save`, `/query`, `/lint`
- Un fichier d'instructions (CLAUDE.md) qui apprend a Claude Code a gerer ton vault
- Un exemple concret pour tester immediatement

## Prerequis

1. **Obsidian** — [obsidian.md](https://obsidian.md) (gratuit)
2. **Claude Code** — installe et fonctionnel
3. **Obsidian CLI** (optionnel) — plugin communautaire de Kepano pour interagir avec le vault

## Installation — 5 minutes

### Etape 1 : Extraire le vault

Dezippe ce dossier ou tu veux sur ta machine. Ce dossier devient ton vault.

### Etape 2 : Ouvrir dans Obsidian

Ouvre Obsidian → "Open folder as vault" → selectionne ce dossier.

Tu verras la structure :

```
raw/           ← Ton espace. Mets tes articles, docs, notes ici.
wiki/          ← L'espace de Claude. Il compile et organise.
CLAUDE.md      ← Les regles du jeu.
skills/        ← Les commandes a installer.
```

### Etape 3 : Installer les skills

Copie les 5 fichiers `.md` du dossier `skills/` (pas le README) dans :

```
~/.claude/commands/
```

Si le dossier `commands/` n'existe pas, cree-le :

```bash
mkdir -p ~/.claude/commands
```

Puis copie les skills :

```bash
cp skills/prime.md skills/ingest.md skills/save.md skills/query.md skills/lint.md ~/.claude/commands/
```

C'est tout.

## Premier test

1. Ouvre un terminal dans le dossier du vault
2. Lance Claude Code (`claude`)
3. Tape `/prime` — Claude charge le contexte du vault
4. Tape `/ingest` — Claude compile l'article exemple dans `raw/notes/`
5. Verifie dans Obsidian : une nouvelle note est apparue dans `wiki/`

## Architecture 3 couches

```
raw/           Humain ecrit, LLM lit (immutable)
   clippings/     Articles web (Obsidian Web Clipper)
   docs/          PDFs, papers, fichiers recus
   notes/         Notes manuscrites, briefs, idees

wiki/          LLM ecrit, Humain browse (compile)
   index.md       Panneau de direction — lu en premier
   log.md         Journal chronologique des operations
   Context/       Profil, objectifs, projets
   Intelligence/  Decisions, recherches, analyses
   Resources/     Templates, patterns, snippets
   Daily/         Journal quotidien auto-genere

CLAUDE.md      Schema — regles du vault (humain configure)
```

## Les 5 commandes

| Commande  | Ce qu'elle fait                               |
| --------- | --------------------------------------------- |
| `/prime`  | Charge le contexte en debut de session        |
| `/ingest` | Compile tes sources brutes en notes wiki      |
| `/save`   | Sauvegarde ce que tu as fait dans la session  |
| `/query`  | Cherche une info dans ton wiki                |
| `/lint`   | Verifie la sante du vault (liens, orphelines) |

## Comment ca marche au quotidien

1. Tu trouves un article interessant → clippe-le dans `raw/clippings/`
2. Lance `/ingest` → Claude le compile dans le wiki
3. Tu bosses sur un projet avec Claude Code → il utilise le wiki comme contexte
4. En fin de session → `/save` pour journaliser
5. De temps en temps → `/lint` pour nettoyer

Le vault grandit avec toi. Plus tu l'alimentes, plus Claude a de contexte, plus il est pertinent.

## C'est incremental

Ce kit est un point de depart, pas un produit fini. Le vault est concu pour evoluer avec ton usage :

- **Semaine 1** — Tu testes avec l'exemple fourni. Tu clippes 2-3 articles. Tu lances /ingest. Le wiki commence a se remplir.
- **Semaine 2-3** — Tu ajoutes tes propres notes, tes docs de projet. Le wiki compile. Les liens entre les notes emergent.
- **Mois 1+** — Le vault devient ton contexte permanent. Claude Code retrouve tes decisions, tes recherches, tes patterns sans que tu te repetes.

Les commandes s'adaptent a la taille du vault. Un vault de 5 notes et un vault de 500 notes utilisent les memes commandes — seul le contenu change.

Tu peux aussi creer tes propres commandes, adapter le CLAUDE.md a tes conventions, ajouter des dossiers dans le wiki. Le kit te donne la fondation, tu construis dessus.

## FAQ

**Ca marche avec quel modele ?**
Tous les modeles Claude via Claude Code. Opus recommande pour la qualite de compilation.

**Ca consomme beaucoup de tokens ?**
L'index.md sert de panneau de direction — Claude ne scanne pas tout le vault a chaque requete. Le cout est minimal.

**Je peux ajouter mes propres commandes ?**
Oui. Cree un fichier `.md` dans `~/.claude/commands/` avec tes instructions.

**C'est compatible avec Obsidian Sync ?**
Oui. Exclue `node_modules/`, `.git/` et autres dossiers techniques dans les parametres de sync.

---

Cree par L'Atelier de l'Automatisation
