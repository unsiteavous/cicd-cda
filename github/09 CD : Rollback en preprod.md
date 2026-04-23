# Étape 9 — CD : rollback preprod

Le rollback restaure l'état exact du serveur avant le dernier déploiement.

## Le script rollback.sh côté serveur

Créez ce fichier dans `/scripts/rollback.sh` sur vos serveurs preprod et prod :

```bash
#!/bin/bash
set -e

APP_DIR="/var/www/monapp"
BACKUP_DIR=$(cat /tmp/last_backup_path)

if [ -z "$BACKUP_DIR" ] || [ ! -d "$BACKUP_DIR" ]; then
  echo "❌ Aucune sauvegarde disponible pour le rollback."
  exit 1
fi

echo "⏪ Rollback depuis $BACKUP_DIR"

# 1. Restaurer la base de données
mysql -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" < "$BACKUP_DIR/db.sql"
# Pour PostgreSQL : psql "$DB_NAME" < "$BACKUP_DIR/db.sql"

# 2. Restaurer les fichiers uploadés
rm -rf "$APP_DIR/uploads"
cp -r "$BACKUP_DIR/uploads" "$APP_DIR/uploads"

# 3. Restaurer le code
rm -rf "$APP_DIR"
cp -r "$BACKUP_DIR/code" "$APP_DIR"

./scripts/post_deploy.sh
echo "✅ Rollback terminé avec succès."
```

## Le job GitHub Actions

Chez GitHub, le déclenchement manuel se fait avec workflow_dispatch :

```yml
# .github/workflows/rollback-preprod.yml
name: Rollback Preprod

on:
  workflow_dispatch:      # déclenchement uniquement via le bouton dans Actions

jobs:
  rollback_preprod:
    runs-on: ubuntu-latest
    environment:
      name: preprod
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
        run: ssh preprod-server "bash /scripts/rollback.sh"
```

- Dans Actions, sélectionnez le workflow Rollback Preprod et cliquez sur Run workflow

- C'est l'équivalent du bouton ▶️ de GitLab

>[!Tip] Point à noter :
>
> Chez GitLab, rollback et déploiement sont dans le même fichier avec `when: manual`. Chez GitHub, on sépare les rollbacks dans des fichiers de workflow dédiés déclenchés par `workflow_dispatch`. C'est plus explicite et plus lisible.