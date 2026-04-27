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

# 2. Restaurer le code
rm -rf "$APP_DIR"
cp -r "$BACKUP_DIR/src" "$APP_DIR"

# 3. Restaurer les fichiers uploadés
rm -rf "$APP_DIR/uploads"
cp -r "$BACKUP_DIR/uploads" "$APP_DIR/uploads"


./scripts/post_deploy.sh
echo "✅ Rollback terminé avec succès."
```

### Le job GitLab

```yaml
rollback_preprod:
  stage: rollback_preprod
  environment:
    name: preprod
  when: on_failure
  only:
    - main
  before_script:
    - *ssh_setup  
  script:
    - | 
      ssh preprod-server "
        cd /var/www/monapp &&
        bash /scripts/rollback.sh
      " 
```

- Dans CI/CD > Pipelines, repérez le bouton ▶️ en face du job rollback_preprod — c'est lui qui déclenche le rollback

- Faites la démonstration en direct : déployez une version cassée, puis rollback

>[!Tip] Point à noter :
>
> Le rollback restaure code + BDD + fichiers en une seule opération atomique. C'est fondamental : si votre nouveau code a ajouté une colonne en BDD, votre ancienne version du code ne saura pas la gérer. Les trois doivent revenir ensemble.