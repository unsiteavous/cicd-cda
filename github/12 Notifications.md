# Étape 12 — Notifications (GitLab)

Les notifications email sont natives sur GitLab — chaque développeur reçoit automatiquement un email en cas d'échec de pipeline, sans aucune configuration requise.

Pour alerter toute l'équipe sur un canal partagé, on configure un webhook Discord, slack, teams ... Voici une proposition pour Discord.

### Créer le webhook Discord

- Dans votre serveur Discord, allez dans Paramètres du salon > Intégrations > Webhooks > Nouveau webhook

- Donnez-lui un nom (ex. GitHub Actions) et choisissez le salon cible

- Copiez l'URL du webhook

- Déclarez-la dans Settings > Secrets and variables > Actions → DISCORD_WEBHOOK_URL

### Le job GitHub Actions

```yml
  notify_failure:
    runs-on: ubuntu-latest
    needs: [lint, test, build, deploy_test, deploy_preprod, deploy_prod]
    if: failure()
    steps:
      - name: Notifier sur Discord
        run: |
          curl -X POST "${{ secrets.DISCORD_WEBHOOK_URL }}" \
            -H "Content-Type: application/json" \
            --data "{
              \"embeds\": [{
                \"title\": \"❌ Pipeline échoué\",
                \"color\": 15158332,
                \"fields\": [
                  { \"name\": \"Projet\", \"value\": \"${{ github.repository }}\", \"inline\": true },
                  { \"name\": \"Branche\", \"value\": \"${{ github.ref_name }}\", \"inline\": true },
                  { \"name\": \"Auteur\", \"value\": \"${{ github.actor }}\", \"inline\": true },
                  { \"name\": \"Lien\", \"value\": \"${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}\" }
                ]
              }]
            }"
```

- `needs:` liste tous les jobs du pipeline — ce job se déclenche si l'un d'entre eux échoue

- `if: failure()` est la condition qui restreint l'exécution aux cas d'échec

- Les variables `${{ github.repository }}`, `${{ github.ref_name }}`, `${{ github.actor }}` sont des variables prédéfinies par GitHub Actions

>[!Tip] Point à noter :
>
> Discord accepte nativement le format embeds de l'API, ce qui permet des messages bien mis en forme sans plugin tiers. Si vous préférez un message simple, remplacez tout le bloc embeds par \"content\": \"❌ Pipeline échoué...\".
> 
> Si vous voulez également notifier en cas de retour au vert (pipeline qui repasse en succès après un échec), dupliquez le job avec `if: success()` et adaptez le message et la couleur (3066993 pour le vert Discord, soit #2ECC71).