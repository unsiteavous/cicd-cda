# Étape 7 — CD : déploiement sur l'environnement de test

## 7.1 — Créer une paire de clés SSH dédiée

```bash
# Sur votre machine locale
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/github_deploy
# Laissez la passphrase vide
```
Cela génère :

- `~/.ssh/github_deploy` → clé privée

- `~/.ssh/github_deploy.pub` → clé publique

## 7.2 — Déposer la clé publique sur chaque serveur

```bash
# Sur le serveur cible
mkdir -p ~/.ssh && chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Collez le contenu de github_deploy.pub
chmod 600 ~/.ssh/authorized_keys
```

Vérifiez que la connexion fonctionne :

```bash
ssh -i ~/.ssh/github_deploy deploy@IP_SERVEUR
```

## 7.3 — Créer le fichier SSH_CONFIG

```text
Host test-server
    HostName 123.45.67.89
    User deploy
    IdentityFile ~/.ssh/githubDeploy
    StrictHostKeyChecking no

Host preprod-server
    HostName 123.45.67.90
    User deploy
    IdentityFile ~/.ssh/githubDeploy
    StrictHostKeyChecking no

Host prod-server
    HostName 123.45.67.91
    User deploy
    IdentityFile ~/.ssh/githubDeploy
    StrictHostKeyChecking no
```

## 7.4 — Encoder les secrets en base64

```bash
# Encoder la clé privée
base64 -w 0 ~/.ssh/github_deploy
# → variable SSH_PRIVATE_KEY
```

## 7.5 — Déclarer les secrets dans GitHub

Allez dans Settings > Secrets and variables > Actions > New repository secret :
| Secret |	Valeur |
|--|--|
| SSH_PRIVATE_KEY |	Résultat du base64 de la clé privée |
|SSH_CONFIG |	Résultat du base64 du fichier config |
|IP_SERVER |	IP ou domaine du serveur |
|DB_USER |	Utilisateur BDD |
|DB_PASS |	Mot de passe BDD |
|DB_NAME |	Nom de la BDD |

>[!Tip] Point à noter :
>
> Chez GitHub, tous les secrets sont automatiquement masqués dans les logs. Il n'y a pas de distinction "masked/protected" comme GitLab — en revanche, vous pouvez restreindre un secret à un environment spécifique (voir étape suivante).

## 7.6 — L'action SSH réutilisable

Chez GitHub, on n'a pas d'ancres YAML natives comme GitLab. On utilise à la place un composite action ou, plus simplement, un bloc `env:` + étapes réutilisées via `uses:`. La solution la plus propre est d'utiliser une action du Marketplace :

```yml
jobs:
  deploy_test:
    runs-on: ubuntu-latest
    environment:
      name: test
      url: https://test.monapp.fr
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

      - name: Déployer
        run: |
          ssh test-server "
            cd /var/www/monapp &&
            git pull origin ${{ github.ref_name }} &&
            ./scripts/post_deploy.sh
          "
```

## 7.7 — Le script post_deploy.sh côté serveur

Cette étape n'est pas obligatoire : vous pouvez bien sûr marquer toutes les actions à faire directement dans vos fichiers workflow, pour tout versionner et tout avoir au même endroit. Dans le cadre de ce cours, comme vous avez toutes et tous des technologies différentes, je l'externalise, mais dans ma pratique professionnelle, je laisse tout dans le fichier `.gitlab-ci.yml` (j'utilise gitlab).

---
Placez-le dans `/scripts/post_deploy.sh` sur chaque serveur :

```bash
#!/bin/bash
set -e

# PHP / Symfony
export APP_ENV=prod; export APP_DEBUG=0;
rm composer.lock;
composer install --no-dev --optimize-autoloader;
rm -f .env.local.php;
composer dump-env prod;
php bin/console doctrine:database:drop --env=prod --force;
php bin/console doctrine:database:create --env=prod;
php bin/console doctrine:migrations:migrate --no-interaction;
php bin/console doctrine:fixtures:load --no-interaction --env=prod;
php bin/console cache:clear --env=prod;
php bin/console cache:warmup --env=prod;
php bin/console lexik:jwt:generate-keypair --skip-if-exists

# Python / Django
pip install -r requirements.txt
python manage.py migrate
sudo systemctl restart gunicorn

# .NET
sudo systemctl restart monapp
```
Le reste du pipeline — la configuration SSH, les stages, les jobs — est identique pour tous les projets.

>[!Tip] Point à noter :
>
> Allez dans Deployments > Environments > test pour voir l'historique des déploiements. Chaque entrée est horodatée, liée au commit exact déployé, et indique qui a déclenché le pipeline.