# Étape 1 — Premier .gitlab-ci.yml

Avant de déployer quoi que ce soit, il faut comprendre comment GitLab lit et exécute vos instructions.

### Le fichier .gitlab-ci.yml

- Il se place à la racine de votre projet

- Il est lu automatiquement par GitLab à chaque push

- Il décrit des stages (étapes) et des jobs (tâches à exécuter)

Commençons par le plus simple possible :

```yaml
stages:
  - hello

hello_world:
  stage: hello
  script:
    - echo "Mon premier pipeline !"
```

- Commitez et pushez ce fichier

- Allez dans CI/CD > Pipelines et regardez votre pipeline s'exécuter en direct

- Cliquez sur le job pour voir les logs en temps réel

## Les GitLab Runners

-  Un Runner est une machine qui exécute vos jobs

- GitLab.com met à disposition des runners mutualisés, gratuits dans une certaine limite

- En entreprise, on héberge souvent ses propres runners (plus de contrôle, pas de limite de minutes)

>[!tip]  Point à noter :
>
> Un job = une série de commandes shell. Tout ce que vous pouvez taper dans un terminal, vous pouvez le mettre dans un job.
>
> Avec gitlab, on fait souvent tout dans un seul fichier. Il est toutefois possible d'éclater le code en sous-parties, dans des fichiers séparés (test.yml, deploy.yml, ...). [Plus d'infos ici](https://blog.stephane-robert.info/docs/pipeline-cicd/gitlab/labs/lab-14-templates-include/).