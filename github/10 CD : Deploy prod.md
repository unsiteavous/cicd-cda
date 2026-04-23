# Étape 10 — CD : prod (manuel) et rollback
## Déploiement prod avec validation humaine

La différence fondamentale avec la preprod repose sur les environnements protégés de GitHub. Dans Settings > Environments > production, activez **Required reviewers** et ajoutez les personnes autorisées à valider.

```yml
  deploy_prod:
    runs-on: ubuntu-latest
    needs: deploy_preprod     # attend que la preprod soit déployée
    environment:
      name: production        # GitHub mettra en pause et demandera une approbation
      url: https://monapp.fr
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Setup SSH
        run: |
          eval $(ssh-agent -s)
          mkdir -p ~/.ssh && chmod 700 ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" | base64 -d > ~/.ssh/githubDeploy
          chmod 400 ~/.ssh/githubDeploy
          echo "${{ secrets.SSH_CONFIG }}" | base64 -d > ~/.ssh/config && chmod 600 ~/.ssh/config
          ssh-add ~/.ssh/githubDeploy
          ssh-keyscan -H ${{ secrets.IP_SERVER }} >> ~/.ssh/known_hosts

      - name: Déployer avec sauvegarde
        run: ssh prod-server "bash /scripts/deploy_with_backup.sh"

Rollback prod

text
# .github/workflows/rollback-prod.yml
name: Rollback Production

on:
  workflow_dispatch:

jobs:
  rollback_prod:
    runs-on: ubuntu-latest
    environment:
      name: production      # déclenche également la validation des reviewers
    steps:
      - name: Setup SSH
        run: |
          eval $(ssh-agent -s)
          mkdir -p ~/.ssh && chmod 700 ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" | base64 -d > ~/.ssh/githubDeploy
          chmod 400 ~/.ssh/githubDeploy
          cat ${{ secrets.SSH_CONFIG }} > ~/.ssh/config && chmod 600 ~/.ssh/config
          ssh-add ~/.ssh/githubDeploy
          ssh-keyscan -H ${{ secrets.IP_SERVER }} >> ~/.ssh/known_hosts

      - name: Rollback
        run: ssh prod-server "bash /scripts/rollback.sh"
```

>[!Tip] Point à noter :
>
> Vous pouvez attacher l'environment production également au workflow de rollback. Ainsi, même le rollback en production nécessite une approbation humaine — on ne revient pas en arrière par accident non plus.