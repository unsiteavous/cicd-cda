# Étape 10 — CD : déploiement prod (manuel) et rollback

La prod est identique à la preprod, à une différence fondamentale : personne ne déploie en production par accident.

### Les jobs

```yaml
deploy_prod:
  stage: deploy_prod
  environment:
    name: production
    url: https://monapp.fr
  only:
    - main
  when: manual     # ← déclenchement humain obligatoire
  before_script:
    - *ssh_setup  
  script:
    - | 
      ssh preprod-server "
        cd /var/www/monapp &&
        bash /scripts/deploy_with_backup.sh
      "

rollback_prod:
  stage: rollback_prod
  environment:
    name: production
  when: manual
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

### Ce que ça change concrètement

- Le pipeline s'arrête après `deploy_preprod` et attend

- Dans CI/CD > Pipelines, le bouton ▶️ apparaît sur `deploy_prod`

- Seules les personnes ayant les droits Maintainer ou Owner peuvent le déclencher

>[!Tip] Point à noter :
>
> Vous n'avez tapé aucune commande à la main sur le serveur de production. Tout est tracé, horodaté, lié à un commit précis, reproductible et annulable. C'est exactement l'objectif de la CD.