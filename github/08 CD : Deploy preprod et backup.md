# Étape 8 — CD : déploiement preprod avec sauvegarde

## Les environnements GitHub

Avant d'aller plus loin, créez vos environnements dans Settings > Environments :

- Créez test, preprod, production

- Pour production, activez **Required reviewers** — cela force une validation humaine avant le déploiement

## Le script `deploy_with_backup.sh` côté serveur

Créez ce fichier dans `/scripts/deploy_with_backup.sh` sur vos serveurs preprod et prod (comme pour les autres, il est possible de tout mettre dans vos workflows):

```bash
#!/bin/bash
set -e    # arrêt immédiat si une commande échoue

APP_DIR="/var/www/monapp"
BACKUP_DIR="/var/backups/monapp/$(date +%Y%m%d_%H%M%S)"

echo "📦 Création de la sauvegarde dans $BACKUP_DIR"
mkdir -p "$BACKUP_DIR"

# 1. Sauvegarde de la base de données
mysqldump -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" > "$BACKUP_DIR/db.sql"
# Pour PostgreSQL : pg_dump "$DB_NAME" > "$BACKUP_DIR/db.sql"

# 2. Sauvegarde des fichiers uploadés par les utilisateurs
cp -r "$APP_DIR/uploads" "$BACKUP_DIR/uploads"

# 3. Sauvegarde de votre base de code -> À COMPLÉTER
cp -r "$APP_DIR/src" "$BACKUP_DIR/src"

# 4. Déploiement du nouveau code
cd "$APP_DIR"
git fetch origin
git checkout main
git fetch origin;
git reset --hard origin/main;
./scripts/post_deploy.sh

# 5. Mémoriser le chemin de sauvegarde pour le rollback
echo "$BACKUP_DIR" > /tmp/last_backup_path
echo "✅ Déploiement terminé. Sauvegarde : $BACKUP_DIR"
```

## Le job GitHub Actions

```yml
jobs:
  deploy_preprod:
    runs-on: ubuntu-latest
    environment:
      name: preprod
      url: https://preprod.monapp.fr
    if: github.ref == 'refs/heads/main'
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

      - name: Déployer avec sauvegarde
        run: ssh preprod-server "bash /scripts/deploy_with_backup.sh"
```

>[!Tip] Point à noter :
>
> `if: github.ref == 'refs/heads/main'` est l'équivalent de `only: - main` chez GitLab. La syntaxe est différente, le principe est identique.