# Étape 4 — CI : tests et coverage

Les tests automatisés sont le cœur de la CI. GitLab peut afficher la couverture directement sur la page du projet.

## Lancer les tests

```text
test:
  stage: test
  script:
    - ./scripts/test.sh
```
Le fichier scripts/test.sh est propre à chaque projet et contient la commande adaptée au langage :

```bash
# Exemple PHP
vendor/bin/phpunit --coverage-cobertura coverage/cobertura.xml

# Exemple Python
pytest --cov=src --cov-report=xml:coverage/cobertura.xml

# Exemple .NET
dotnet test --collect:"XPlat Code Coverage"
```

## Afficher la coverage dans GitLab

```text
test:
  stage: test
  script:
    - ./scripts/test.sh
  coverage: '/Lines:\s*(\d+\.\d+)%/'     # regex à adapter ⁽¹⁾ selon votre outil
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura.xml
```
[1] https://docs.gitlab.com/ci/testing/code_coverage/#coverage-regex-patterns

- La couverture apparaît dans CI/CD > Pipelines, sur chaque job, et sous forme de badge sur la page du projet

- Vous pouvez rendre le job bloquant si la coverage descend sous un seuil : ajoutez `--fail-under=80` à votre commande de test



>[!tip] Point à noter :
>
> Le coverage seul ne garantit pas des bons tests. Un code à 100% de coverage peut très bien ne tester que des return true. Elle reste cependant un indicateur utile de discipline d'équipe.