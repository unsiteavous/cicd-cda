# Étape 4 — CI : tests et coverage

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Lancer les tests
        run: ./scripts/test.sh

      - name: Publier le rapport de coverage
        uses: codecov/codecov-action@v4    # ou coverallsapp/github-action
        with:
          files: coverage/cobertura.xml
          fail_ci_if_error: true
```

Le fichier `scripts/test.sh` contient la commande adaptée au langage :

```bash
# PHP
vendor/bin/phpunit --coverage-clover coverage/cobertura.xml

# Python
pytest --cov=src --cov-report=xml:coverage/cobertura.xml

# .NET
dotnet test --collect:"XPlat Code Coverage"
```
>[!Tip] Point à noter :
>
> GitHub n'affiche pas nativement la coverage dans l'interface comme GitLab. On passe par un service tiers comme Codecov ou Coveralls, qui génèrent un badge et des rapports détaillés. Les deux ont une offre gratuite pour les projets publics.