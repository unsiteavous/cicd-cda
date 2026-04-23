# Étape 0 — Mise en place (prérequis)

Avant d'écrire la moindre ligne de CI/CD, posons les bases :

## Côté GitHub

- Avoir un projet existant (ou en créer un de démo avec un simple hello world)

- Maîtriser l'interface GitHub :

  - Actions : historique des workflows et des runs

  - Environments : gestion des environnements (test, preprod, prod)

  - Settings > Secrets and variables > Actions : variables secrètes

  - Settings > Branches : règles de protection des branches

  - Releases : historique des versions

- Déclarer les secrets : `SSH_PRIVATE_KEY`, `SSH_CONFIG`, `IP_SERVER`, `DB_USER`, `DB_PASS`, `DB_NAME`, `SEMANTIC_RELEASE_TOKEN`, ...

## Côté serveurs

- Avoir un serveur avec accès SSH

- Avoir fait un déploiement à la main, pour se rendre compte

- Structure des dossiers côté serveur : `/var/www/monapp`, `/var/backups/monapp`, `/scripts/`

- Le git clone initial en SSH sur chaque serveur, et la clé SSH du Runner dans authorized_keys

>[!Tip] Point à noter :
>
> Chez GitHub, les secrets se déclarent dans Settings > Secrets and variables > Actions. Contrairement à GitLab, GitHub distingue les repository secrets (pour un seul dépôt) des organization secrets (partagés entre plusieurs dépôts d'une organisation).