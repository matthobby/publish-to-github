---
name: publish-to-github
description: "Transforme une skill Claude existante en repo GitHub installable comme marketplace/plugin dans Claude Desktop. Utilise cette skill dès que l'utilisateur veut publier, partager, exporter ou mettre sur GitHub une skill — même s'il dit juste 'je veux mettre ma skill sur GitHub', 'rendre ma skill installable', 'packager mon plugin', 'créer un repo pour ma skill', ou 'publier ma skill'. Couvre aussi les mises à jour d'un repo existant après modification de la skill."
---

# Publish to GitHub

Cette skill transforme n'importe quelle skill Claude (un dossier contenant un SKILL.md et éventuellement des evals) en un **repo GitHub structuré comme marketplace** — le format exact que Claude Desktop attend quand on ajoute une URL dans Paramètres > Personnaliser > Plugins.

## Pourquoi ce format

Claude Desktop ne permet pas d'installer un plugin brut depuis GitHub. Il faut un **marketplace** — un repo qui contient un `marketplace.json` listant un ou plusieurs plugins. C'est comme ça que fonctionnent les marketplaces existants (Jean-Claude-Marketplace, knowledge-work-plugins, etc.).

La structure cible est :

```
repo/
├── .claude-plugin/
│   └── marketplace.json       ← Déclare le marketplace et liste les plugins
├── plugins/
│   └── <nom-skill>/
│       ├── .claude-plugin/
│       │   └── plugin.json    ← Déclare le plugin individuel
│       └── skills/
│           └── <nom-skill>/
│               ├── SKILL.md   ← La skill elle-même
│               └── evals/
│                   └── evals.json
├── README.md
└── LICENSE (optionnel)
```

## Workflow

### Étape 1 — Identifier la skill source

Demande à l'utilisateur quelle skill il veut publier. Les skills se trouvent généralement dans :
- Le dossier courant de travail
- `/mnt/.skills/skills/` (skills installées)
- Un chemin que l'utilisateur indique

Ce dont tu as besoin au minimum :
- Le fichier `SKILL.md` (obligatoire)
- Le dossier `evals/` s'il existe (recommandé)
- Tout dossier `references/`, `scripts/`, `assets/` bundlé avec la skill

Lis le frontmatter YAML du SKILL.md pour extraire `name` et `description`. Ce sont les identifiants de base du plugin.

### Étape 2 — Collecter les infos manquantes

Tu as besoin de quelques infos supplémentaires. Si tu peux les déduire du contexte ou de la conversation précédente, fais-le — sinon demande :

- **Nom du repo GitHub** (ex: `veille-youtube`, `pdf-merger`). Par défaut, utilise le `name` du frontmatter.
- **Nom d'auteur** pour le plugin.json (ex: `MattEpicAdventures`). Vérifie si le nom de l'utilisateur est disponible dans le contexte.
- **URL du repo GitHub** si elle existe déjà (ex: `https://github.com/user/repo`). Si l'utilisateur n'a pas encore créé le repo, note-le — tu lui donneras les instructions à la fin.
- **Description courte** pour le marketplace. Par défaut, réutilise la description du SKILL.md mais en version condensée (1-2 phrases, sans accents dans le JSON pour éviter les problèmes d'encodage).

### Étape 3 — Créer la structure

Crée le dossier complet dans le répertoire de travail de l'utilisateur (typiquement `mnt/Documents/`).

```bash
REPO_DIR="mnt/Documents/<nom-repo>"
SKILL_NAME="<nom-skill>"

mkdir -p "$REPO_DIR/.claude-plugin"
mkdir -p "$REPO_DIR/plugins/$SKILL_NAME/.claude-plugin"
mkdir -p "$REPO_DIR/plugins/$SKILL_NAME/skills/$SKILL_NAME/evals"
```

#### marketplace.json

Crée `.claude-plugin/marketplace.json` à la racine :

```json
{
  "name": "<nom-repo>",
  "metadata": {
    "description": "Description courte du marketplace."
  },
  "owner": {
    "name": "<nom-auteur>"
  },
  "plugins": [
    {
      "name": "<nom-skill>",
      "source": "./plugins/<nom-skill>",
      "description": "Description du plugin sans accents."
    }
  ]
}
```

Points importants :
- Le champ `name` du marketplace peut être identique au nom du repo
- Le champ `source` doit pointer vers `./plugins/<nom-skill>` (chemin relatif)
- La description dans `plugins[]` devrait être sans accents pour maximiser la compatibilité

#### plugin.json

Crée `plugins/<nom-skill>/.claude-plugin/plugin.json` :

```json
{
  "name": "<nom-skill>",
  "description": "Description complète du plugin.",
  "version": "1.0.0"
}
```

Garde le plugin.json minimaliste — seulement `name`, `description`, `version`. C'est le format qui marche (validé sur les marketplaces existants). Évite d'ajouter des champs comme `author`, `repository`, `keywords`, `license` — ils ne sont pas nécessaires et pourraient causer des problèmes de parsing.

#### Copie des fichiers de la skill

Copie tous les fichiers de la skill source dans `plugins/<nom-skill>/skills/<nom-skill>/` :

```bash
cp source/SKILL.md "$REPO_DIR/plugins/$SKILL_NAME/skills/$SKILL_NAME/"
cp -r source/evals/ "$REPO_DIR/plugins/$SKILL_NAME/skills/$SKILL_NAME/evals/" 2>/dev/null
cp -r source/references/ "$REPO_DIR/plugins/$SKILL_NAME/skills/$SKILL_NAME/references/" 2>/dev/null
cp -r source/scripts/ "$REPO_DIR/plugins/$SKILL_NAME/skills/$SKILL_NAME/scripts/" 2>/dev/null
cp -r source/assets/ "$REPO_DIR/plugins/$SKILL_NAME/skills/$SKILL_NAME/assets/" 2>/dev/null
```

#### README.md

Génère un README.md concis à la racine :

```markdown
# <Nom lisible du marketplace>

<Description en 1-2 phrases>

## Plugin : <nom-skill>

<Description du plugin>

## Installation

Dans Claude Desktop :
1. **Paramètres > Personnaliser > Plugins**
2. Ajouter l'URL : `https://github.com/<user>/<repo>`
3. Le plugin apparaît dans tes skills

## Structure

[Arbre des fichiers]

## Licence

MIT
```

### Étape 4 — Initialiser git et préparer le push

```bash
cd "$REPO_DIR"
git init
git add -A
git commit -m "v1.0 — <nom-skill> marketplace plugin"
```

Puis donne à l'utilisateur les commandes pour créer le repo sur GitHub et push :

```
1. Va sur github.com/new et crée le repo "<nom-repo>"
2. Puis dans ton terminal :
   cd ~/Documents/<nom-repo>
   git remote add origin https://github.com/<user>/<repo>.git
   git push -u origin main
3. Dans Claude Desktop : Paramètres > Personnaliser > Plugins > ajoute https://github.com/<user>/<repo>
```

### Étape 5 — Vérification

Après le push, vérifie avec l'utilisateur que :
- Le repo est visible sur GitHub
- L'ajout dans Claude Desktop fonctionne (pas de "Échec de l'ajout du marketplace")
- La skill apparaît dans les skills disponibles

Si l'utilisateur a une erreur "Échec de l'ajout du marketplace", les causes les plus fréquentes sont :
- Le `marketplace.json` n'est pas dans `.claude-plugin/` à la racine du repo
- Le champ `source` dans `plugins[]` ne pointe pas vers le bon chemin
- Le `plugin.json` du plugin manque ou contient des champs non supportés
- Le repo GitHub n'est pas public

## Mise à jour d'un repo existant

Si l'utilisateur a déjà un repo marketplace et veut mettre à jour sa skill :

1. Identifie le repo local (demande le chemin ou cherche dans `mnt/Documents/`)
2. Copie les fichiers mis à jour de la skill dans `plugins/<nom>/skills/<nom>/`
3. **TOUJOURS bumper la version dans `plugin.json`** — c'est ce que Claude Desktop utilise pour détecter les mises à jour. Sans ce bump, l'utilisateur restera sur l'ancienne version même après un push.

   Pour bumper la version, lis le `plugin.json` actuel, parse la version semver (ex: `1.0.0`), et incrémente :
   - **Patch** (1.0.0 → 1.0.1) : corrections mineures, typos
   - **Minor** (1.0.0 → 1.1.0) : nouvelles fonctionnalités, ajouts de sections
   - **Major** (1.0.0 → 2.0.0) : refonte complète, changements breaking

   En cas de doute, utilise minor. Exemple en Python :

   ```python
   import json
   with open(plugin_json_path, 'r') as f:
       data = json.load(f)
   parts = data['version'].split('.')
   parts[1] = str(int(parts[1]) + 1)  # bump minor
   parts[2] = '0'  # reset patch
   data['version'] = '.'.join(parts)
   with open(plugin_json_path, 'w') as f:
       json.dump(data, f, indent=2, ensure_ascii=False)
   ```

4. Stage, commit et donne les commandes pour push
5. Indique à l'utilisateur d'aller dans **Paramètres > Personnaliser > Plugins** et de rafraîchir le marketplace pour que Claude récupère la nouvelle version

## Ajout d'un second plugin à un marketplace existant

Si l'utilisateur veut ajouter une autre skill à son marketplace :

1. Crée le dossier `plugins/<nouveau-nom>/` avec la même structure
2. Ajoute une entrée dans le tableau `plugins` de `marketplace.json`
3. Met à jour le README
4. Commit et push

## Erreurs fréquentes à éviter

- **Ne pas confondre plugin et marketplace** : un repo avec juste `.claude-plugin/plugin.json` (sans `marketplace.json`) ne peut pas être ajouté comme source dans Claude Desktop. Il faut toujours le wrapper marketplace.
- **Ne pas mettre de caractères spéciaux dans les JSON** : évite les accents dans les descriptions des fichiers JSON (marketplace.json, plugin.json) pour la compatibilité.
- **Garder plugin.json minimal** : seulement `name`, `description`, `version`. Pas de champs en plus.
- **Chemins relatifs dans source** : toujours `./plugins/nom-skill`, jamais de chemin absolu.
