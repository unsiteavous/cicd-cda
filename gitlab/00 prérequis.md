# Étape 0 — Mise en place (prérequis)

Avant d'écrire la moindre ligne de CI/CD, posons les bases :

## Côté GitLab

- Avoir un projet existant (ou en créer un de démo avec un simple hello world)

- Maitriser l'interface gitlab :
  -  CI/CD > Pipelines,
  -  Deployments > Environments,
  -  Settings > CI/CD > Variables
  -  Planification > Tableau des tickets
  -  Settings > CI/CD > Access Token
  -  ...

- Déclarer les variables secrètes : SSH_PRIVATE_KEY, TEST_SERVER, PREPROD_SERVER, PROD_SERVER, DB_USER, DB_PASS, DB_NAME, ...

## Côté serveurs

- Avoir un serveur avec accès SSH

- Avoir fait un déploiement à la main, pour se rendre compte

- Structure des dossiers côté serveur : /var/www/monapp, /var/backups/monapp, /scripts/

- Le git clone initial en SSH sur chaque serveur, et la clé SSH du Runner dans authorized_keys

>[!tip] Point à noter :
> 
> Ce que vous faites à la main maintenant, le pipeline va le faire à votre place

