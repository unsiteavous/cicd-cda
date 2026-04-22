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
    "@semantic-release/gitlab"
  ]
}
```

### Rôle de chaque plugin :

- commit-analyzer : lit vos commits et détermine le type de version (patch/minor/major)

- release-notes-generator : génère les notes de release à partir des commits

- changelog : met à jour le fichier CHANGELOG.md dans votre dépôt

- gitlab : crée le tag et la release directement dans GitLab

## 11.3 — La variable SEMANTIC_RELEASE_TOKEN

**semantic-release** a besoin de droits pour créer des tags, des releases, et mettre à jour `CHANGELOG.md` sur votre projet.

Dans GitLab, allez dans Settings > Access Tokens et créez un token avec les scopes api et write_repository. Déclarez-le ensuite dans Settings > CI/CD > Variables :

| Variable |	Valeur |	Protected |	Masked |
|--|--|--|--|
| SEMANTIC_RELEASE_TOKEN |	Votre access token |	✅ |	✅ |

## 11.4 — Le job GitLab

```yaml
release:
  image: node:lts-slim
  stage: release
  variables:
    GL_TOKEN: $SEMANTIC_RELEASE_TOKEN          # nom attendu par le plugin @semantic-release/gitlab
    GIT_AUTHOR_NAME: "Robot Semantic Release"
    GIT_AUTHOR_EMAIL: "semantic@release.ci"
    GIT_COMMITTER_NAME: "Robot Semantic Release"
    GIT_COMMITTER_EMAIL: "semantic@release.ci"
  before_script:
    - apt-get update && apt-get install -y --no-install-recommends git-core ca-certificates
    - npm install -g semantic-release @semantic-release/gitlab @semantic-release/changelog
    - git remote set-url origin "https://gitlab-ci-token:${GL_TOKEN}@${CI_SERVER_HOST}/${CI_PROJECT_PATH}.git"
  script:
    - semantic-release
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

### Détail des points importants :

- `GL_TOKEN` : le plugin @semantic-release/gitlab attend spécifiquement cette variable pour s'authentifier

- `GIT_AUTHOR_NAME` / `GIT_COMMITTER_NAME` : identifient le "robot" qui va commiter le CHANGELOG.md — permet de distinguer visuellement les commits automatiques des commits humains dans l'historique

- `git remote set-url` : reconfigure l'URL du dépôt pour utiliser le token au lieu de SSH — nécessaire pour que semantic-release puisse pousser le changelog et les tags

- `$CI_SERVER_HOST` et` $CI_PROJECT_PATH` : variables prédéfinies par GitLab — elles s'adaptent automatiquement à n'importe quel projet, sans données en dur

   `git-core ca-certificates` : `git-core` pour lire l'historique des commits, `ca-certificates` pour les connexions HTTPS vers GitLab

## 11.5 — Ce que ça produit dans GitLab

Une fois le job exécuté :

- Un tag Git est créé automatiquement (v1.3.0) dans Repository > Tags
- Une release GitLab est générée avec les notes de version dans Deployments > Releases
- Le fichier CHANGELOG.md est mis à jour dans votre dépôt par le robot

>[!Tip] Point à noter :
>
> Le commit de mise à jour du `CHANGELOG.md` est signé au nom du "Robot Semantic Release". 

