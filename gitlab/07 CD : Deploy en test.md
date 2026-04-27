# Étape 7 — CD : déploiement sur l'environnement de test

Avant de déployer, il faut mettre en place la connexion SSH entre le Runner GitLab et vos serveurs. C'est une étape de configuration à faire une seule fois, mais il faut la comprendre en détail.

## 7.1 — Créer une paire de clés SSH dédiée

On ne réutilise pas sa clé SSH personnelle. On crée une clé dédiée au déploiement, sans passphrase (le Runner ne peut pas saisir de mot de passe interactif).

```bash
# Sur votre machine locale
ssh-keygen -t ed25519 -C "gitlab-runner-deploy" -f ~/.ssh/gitlab_deploy
# Laissez la passphrase vide (appuyez deux fois sur Entrée)
```
Cela génère deux fichiers :

- `~/.ssh/gitlab_deploy` → clé privée (ne la partagez jamais)

- `~/.ssh/gitlab_deploy.pub` → clé publique (à déposer sur les serveurs)

## 7.2 — Déposer la clé publique sur chaque serveur

Sur chaque serveur (test, preprod, prod), connectez-vous et ajoutez la clé publique à l'utilisateur de déploiement :

```bash
# Sur le serveur cible, connecté en tant que deploy (ou root)
mkdir -p ~/.ssh && chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Collez le contenu de gitlab_deploy.pub, sauvegardez
chmod 600 ~/.ssh/authorized_keys
```
Vérifiez que la connexion fonctionne depuis votre machine :

```bash
ssh -i ~/.ssh/gitlab_deploy deploy@IP_SERVEUR
```
>[!Tip] Point à noter :
>
> L'utilisateur deploy doit avoir les droits nécessaires sur `/var/www/monapp` et `/scripts/`, mais pas les droits root. Principe du moindre privilège.

## 7.3 — Créer le fichier SSH_CONFIG

Le fichier `~/.ssh/config` permet de définir des alias de connexion : au lieu d'écrire `ssh -i ~/.ssh/gitlab_deploy deploy@123.45.67.89`, on écrit juste `ssh mon-serveur-test`. C'est ce fichier qu'on va stocker dans GitLab.

```text
Host test-server
    HostName 123.45.67.89        # ou le nom de domaine
    User deploy
    IdentityFile ~/.ssh/epiManager
    StrictHostKeyChecking no

Host preprod-server
    HostName 123.45.67.90
    User deploy
    IdentityFile ~/.ssh/epiManager
    StrictHostKeyChecking no

Host prod-server
    HostName 123.45.67.91
    User deploy
    IdentityFile ~/.ssh/epiManager
    StrictHostKeyChecking no
```

## 7.4 — Encoder les secrets en base64

Le Runner GitLab va récupérer la clé privée depuis une variable. Mais une clé privée contient des sauts de ligne et des caractères spéciaux qui peuvent être altérés lors du passage en variable d'environnement. On l'encode en base64 pour la transmettre de façon fiable :

```bash
# Sur votre machine locale

# Encoder la clé privée
base64 -w 0 ~/.ssh/gitlab_deploy
# Copiez le résultat → ce sera la variable SSH_PRIVATE_KEY
```
>[!Tip] Point à noter :
>
> `-w 0` supprime les retours à la ligne dans la sortie base64. Sans ça, GitLab tronquerait la variable. Le script de déploiement décodera avec `base64 -d` pour retrouver le fichier original.

## 7.5 — Déclarer les variables dans GitLab

Allez dans Settings > CI/CD > Variables et ajoutez :

| Variable	| Valeur	| Type	| Protected	| Masked |
|--|--|--|--|--|
|SSH_PRIVATE_KEY	|Résultat du base64 de la clé privée	|Variable	|✅	|✅|
|SSH_CONFIG	|Contenu du fichier config	|Fichier|	✅|	❌|
|IP_SERVER	|IP ou domaine du serveur (pour ssh-keyscan)	|Variable|	✅	|❌|
|DB_USER|	Utilisateur BDD	|Variable	|✅	|✅|
|DB_PASS|	Mot de passe BDD	|Variable	|✅	|✅|
|DB_NAME|	Nom de la BDD	|Variable	|✅	|❌|

Vous aurez certainement plus de variables que ça, certaines sont à multiplier par le nombre d'environnements de déploiement que vous aurez : base de données notamment.

>[!warning] Rappel :
> 
> **Protected** : la variable n'est accessible que sur les branches protégées (main, develop, ...) Restrictif, à vous de voir.
>
> **Masked** : la valeur n'apparaît jamais en clair dans les logs du pipeline

## 7.6 — L'ancre SSH dans .gitlab-ci.yml

Plutôt que de répéter la configuration SSH dans chaque job de déploiement, on utilise une ancre YAML. C'est une fonctionnalité native de YAML qui permet de définir un bloc une fois et de le réutiliser partout :

```yaml
# =========================
# SSH SETUP ANCHOR
# =========================
.ssh_setup: &ssh_setup
  - eval $(ssh-agent -s)
  - mkdir -p ~/.ssh && chmod 700 ~/.ssh
  - cat $SSH_PRIVATE_KEY | base64 -d > ~/.ssh/monSite
  - chmod 400 ~/.ssh/monSite
  - cat $SSH_CONFIG > ~/.ssh/config && chmod 600 ~/.ssh/config
  - ssh-add ~/.ssh/monSite
  - ssh-keyscan -H $IP_SERVER >> ~/.ssh/known_hosts
```
Détail ligne par ligne :

- eval $(ssh-agent -s) : démarre l'agent SSH qui va gérer les clés en mémoire

- base64 -d : décode la clé privée et le fichier config stockés en base64

- chmod 400 : permissions strictes sur la clé privée (SSH refuse de l'utiliser sinon)

- ssh-add : charge la clé dans l'agent SSH

- ssh-keyscan : récupère et mémorise l'empreinte du serveur pour éviter le prompt de confirmation au premier accès

## 7.7 — Le job de déploiement

On utilise l'ancre avec `<<: *ssh_setup` pour injecter toute la configuration SSH dans le job :

```yaml
deploy_test:
  stage: deploy_test
  environment:
    name: test
    url: https://test.monapp.fr
  before_script:
    - *ssh_setup               # injection de l'ancre
  script:
    - | 
      ssh test-server "
        cd /var/www/monapp &&
        git fetch origin &&
        git reset --hard origin/$CI_COMMIT_BRANCH &&
        ./scripts/post_deploy.sh
      "
```

Grâce au fichier `~/.ssh/config`, `ssh test-server` suffit : l'IP, l'utilisateur et la clé sont déjà configurés.

## 7.8 — Le script post_deploy.sh côté serveur

Cette étape n'est pas obligatoire : vous pouvez bien sûr marquer toutes les actions à faire directement dans votre fichier `.gitlab-ci.yml`, pour tout versionner et tout avoir au même endroit. Dans le cadre de ce cours, comme vous avez toutes et tous des technologies différentes, je l'externalise, mais dans ma pratique professionnelle, je laisse tout dans le fichier `.gitlab-ci.yml`.

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