# Étape 1 — Premier workflow GitHub Actions

- Chez GitHub, il n'y a pas de .gitlab-ci.yml — les workflows sont des fichiers .yml placés dans .github/workflows/.

## La structure de base

```yml
# .github/workflows/pipeline.yml
name: CI/CD Pipeline

on:
  push:
    branches:
      - "**"          # tous les pushs déclenchent le pipeline
  workflow_dispatch:  # déclenchement manuel depuis l'interface GitHub

jobs:
  hello_world:
    runs-on: ubuntu-latest
    steps:
      - name: Mon premier workflow
        run: echo "Mon premier pipeline !"
```

- Commitez et pushez ce fichier dans .github/workflows/

- Allez dans Actions et regardez votre workflow s'exécuter en direct

- Cliquez sur le job pour voir les logs en temps réel

## Les GitHub Runners

GitHub met à disposition des runners mutualisés (ubuntu-latest, windows-latest, macos-latest), gratuits dans une certaine limite.

En entreprise, on héberge ses propres runners (self-hosted runners), déclarés dans Settings > Actions > Runners.

>[!Tip] Point à noter :
>
> Un job = une série de commandes shell. Tout ce que vous pouvez taper dans un terminal, vous pouvez le mettre dans un job.
>
> AvecGitHub, vous pouvez avoir plusieurs fichiers dans .github/workflows/ — un par grande thématique (ci.yml, deploy.yml, release.yml...). C'est plus modulaire, mais demande plus d'organisation.