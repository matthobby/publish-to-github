# Publish to GitHub

Plugin Claude pour transformer n'importe quelle skill en repo GitHub installable dans Claude Desktop.

## Le problème

Quand tu crées une skill Claude, elle fonctionne localement. Mais pour la partager ou l'installer via l'interface Plugins de Claude Desktop, il faut une structure **marketplace** précise — avec `marketplace.json`, `plugin.json`, et une arborescence spécifique. C'est pas évident à deviner.

## Ce que fait ce plugin

Tu lui dis "mets ma skill sur GitHub" et il :

1. Lit ta skill (SKILL.md + evals + references)
2. Crée toute la structure marketplace
3. Initialise git
4. Te donne les commandes pour push

Il gère aussi les **mises à jour** et l'**ajout de plusieurs plugins** dans un même marketplace.

## Installation

Dans Claude Desktop :
1. **Paramètres > Personnaliser > Plugins**
2. Ajouter l'URL : `https://github.com/matthobby/publish-to-github`
3. Le plugin apparaît dans tes skills

## Utilisation

Dis simplement :
- "Je veux mettre ma skill sur GitHub"
- "Publie mon plugin"
- "Rends ma skill installable"
- "Package ma skill pour la partager"

## Structure

```
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── publish-to-github/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── publish-to-github/
│               ├── SKILL.md
│               └── evals/
│                   └── evals.json
└── README.md
```

## Licence

MIT
