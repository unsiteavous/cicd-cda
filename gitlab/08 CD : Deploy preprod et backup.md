# Étape 8 — CD : déploiement preprod avec sauvegarde

Avant chaque déploiement sur preprod et prod, on sauvegarde tout. C'est ce qui rend le rollback possible.

##Le script `deploy_with_backup.sh` côté serveur

Créez ce fichier dans `/scripts/deploy_with_backup.sh` sur vos serveurs preprod et prod (comme pour les autres, il est possible de tout mettre dans gitlab-ci):

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

# 3. Déploiement du nouveau code
cd "$APP_DIR"
git fetch origin
git checkout main
git pull origin main
./scripts/post_deploy.sh

# 4. Mémoriser le chemin de sauvegarde pour le rollback
echo "$BACKUP_DIR" > /tmp/last_backup_path
echo "✅ Déploiement terminé. Sauvegarde : $BACKUP_DIR"
```

### Le job GitLab

```yaml
deploy_preprod:
  stage: deploy_preprod
  environment:
    name: preprod
    url: https://preprod.monapp.fr
  only:
    - main
  before_script:
    - *ssh_setup  
  script:
    - | 
      ssh preprod-server "
        cd /var/www/monapp &&
        bash /scripts/deploy_with_backup.sh
      "
```
>[!Tip] Point à noter :
> Commencer une ligne avec `|` dans les scripts précise qu'on va écrire sur plusieurs lignes. Sans ça, les retours à la lignes ne seraient pas interprétés.
> 
> `set -e` en tête de script bash est crucial : si la sauvegarde échoue, le déploiement ne se lance pas. Sans ça, on pourrait déployer sans sauvegarde valide et se retrouver sans possibilité de rollback.