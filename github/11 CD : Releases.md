# Étape 11 — Releases avec semantic-release
Les releases permettent de versionner votre code automatiquement, en s'appuyant sur vos Conventional Commits de l'étape 2.

## 11.1 Comment ça fonctionne

semantic-release analyse vos messages de commit depuis la dernière release et détermine automatiquement le prochain numéro de version selon SemVer :
| Type de commit | 	Impact sur la version | 
|--|--|
| fix: |	Patch → 1.2.3 → 1.2.4 |
| feat: |	Minor → 1.2.3 → 1.3.0 |
| feat!: ou BREAKING CHANGE	| Major → 1.2.3 → 2.0.0 |
| chore:, docs:, style:	 | Aucune release créée |


>[!Tip] Point à noter :
>
> Si vous n'avez pas respecté les Conventional Commits, semantic-release ne créera aucune release. C'est un excellent moyen de comprendre concrètement pourquoi cette discipline a de la valeur.


## 11.2 — Le fichier .releaserc.json

Placez ce fichier à la racine de votre projet :

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/github"
  ]
}
```
### Rôle de chaque plugin :

- commit-analyzer : lit vos commits et détermine le type de version (patch/minor/major)

- release-notes-generator : génère les notes de release à partir des commits

- changelog : met à jour le fichier CHANGELOG.md dans votre dépôt

- github : crée le tag et la release directement dans Github

## 11.3 — La variable SEMANTIC_RELEASE_TOKEN

**semantic-release** a besoin de droits pour créer des tags, des releases, et mettre à jour `CHANGELOG.md` sur votre projet.

Dans GitHub, allez dans Settings > Developer settings > Personal access tokens > Fine-grained tokens et créez un token avec les permissions contents: write et issues: write. Déclarez-le dans Settings > Secrets and variables > Actions :

|Secret|	Protected|	Masked|
|--|--|--|
|SEMANTIC_RELEASE_TOKEN	|✅|✅|


## 11.4 — Le job GitHub Actions

```yml
release:
  runs-on: ubuntu-latest
  needs: [deploy_preprod]   # ou deploy_prod selon votre choix
  if: github.ref == 'refs/heads/main'
  env:
    GITHUB_TOKEN: ${{ secrets.SEMANTIC_RELEASE_TOKEN }}   # nom attendu par le plugin @semantic-release/github
    GIT_AUTHOR_NAME: "Robot Semantic Release"
    GIT_AUTHOR_EMAIL: "semantic@release.ci"
    GIT_COMMITTER_NAME: "Robot Semantic Release"
    GIT_COMMITTER_EMAIL: "semantic@release.ci"
  steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0       # obligatoire : semantic-release a besoin de tout l'historique Git
        persist-credentials: false

    - name: Installer semantic-release
      run: npm install -g semantic-release @semantic-release/github @semantic-release/changelog

    - name: Créer la release
      run: semantic-release
```

### Détail des points importants :

- `GITHUB_TOKEN` : le plugin @semantic-release/github attend spécifiquement cette variable pour s'authentifier

- `GIT_AUTHOR_NAME` / `GIT_COMMITTER_NAME` : identifient le "robot" qui va commiter le CHANGELOG.md — permet de distinguer visuellement les commits automatiques des commits humains dans l'historique

- `fetch-depth: 0` est obligatoire : par défaut, actions/checkout ne récupère que le dernier commit. semantic-release a besoin de tout l'historique pour analyser les commits depuis la dernière release

- `persist-credentials: false` évite les conflits entre les credentials du checkout et ceux de semantic-release

## 11.5 — Ce que ça produit dans GitHub

Un tag Git est créé automatiquement (v1.3.0) dans Code > Tags

Une GitHub Release est générée avec les notes de version dans Releases

Le fichier CHANGELOG.md est mis à jour dans votre dépôt par le robot