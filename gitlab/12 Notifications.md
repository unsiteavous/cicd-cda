# Étape 12 — Notifications (GitLab)

Les notifications email sont natives sur GitLab — chaque développeur reçoit automatiquement un email en cas d'échec de pipeline, sans aucune configuration requise.

Pour alerter toute l'équipe sur un canal partagé, on configure un webhook Discord, slack, teams ... Voici une proposition pour Discord.

### Créer le webhook Discord

- Dans votre serveur Discord, allez dans Paramètres du salon > Intégrations > Webhooks > Nouveau webhook

- Donnez-lui un nom (ex. GitLab CI) et choisissez le salon cible

- Copiez l'URL du webhook

- Déclarez-la dans Settings > CI/CD > Variables → DISCORD_WEBHOOK_URL (Masked ✅)

### Le job GitLab

```yml
notify_failure:
  stage: .post          # stage spécial GitLab, toujours exécuté en dernier
  when: on_failure
  script:
    - |
      curl -X POST "$DISCORD_WEBHOOK_URL" \
        -H "Content-Type: application/json" \
        --data "{
          \"embeds\": [{
            \"title\": \"❌ Pipeline échoué\",
            \"color\": 15158332,
            \"fields\": [
              { \"name\": \"Projet\", \"value\": \"$CI_PROJECT_NAME\", \"inline\": true },
              { \"name\": \"Branche\", \"value\": \"$CI_COMMIT_BRANCH\", \"inline\": true },
              { \"name\": \"Auteur\", \"value\": \"$CI_COMMIT_AUTHOR\", \"inline\": true },
              { \"name\": \"Lien\", \"value\": \"$CI_PIPELINE_URL\" }
            ]
          }]
        }"
```

- `stage: .post` est un stage spécial réservé par GitLab — il s'exécute toujours après tous les autres stages, même en cas d'échec

- `when: on_failure` restreint ce job aux pipelines qui ont échoué

- Les variables `$CI_PROJECT_NAME`, `$CI_COMMIT_BRANCH`, `$CI_COMMIT_AUTHOR`, `$CI_PIPELINE_URL` sont des variables prédéfinies par GitLab

>[!Tip] Point à noter :
>
> Discord accepte nativement le format embeds de l'API, ce qui permet des messages bien mis en forme sans plugin tiers. Si vous préférez un message simple, remplacez tout le bloc embeds par \"content\": \"❌ Pipeline échoué...\".
> 
> Si vous voulez également notifier en cas de retour au vert (pipeline qui repasse en succès après un échec), dupliquez le job avec `when: on_success` et adaptez le message et la couleur (3066993 pour le vert Discord, soit #2ECC71).