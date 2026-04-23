# Étape 3 — CI : linting et analyse statique

```yml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4     # récupère le code source

      # PHP
      - name: Lint PHP
        run: |
          find . -name "*.php" -exec php -l {} \;
          composer install --no-interaction
          vendor/bin/phpstan analyse src/ --level=5

      # Python
      - name: Lint Python
        run: |
          pip install pylint
          pylint src/ --fail-under=8.0

      # JavaScript
      - name: Lint JS
        run: |
          npm ci
          npm run lint
```

- `actions/checkout@v4` est une action GitHub — l'équivalent des image: Docker de GitLab. C'est une brique réutilisable publiée sur le GitHub Marketplace

- Les jobs d'un même workflow s'exécutent en parallèle par défaut. Pour les chaîner, utilisez `needs:`

>[!Tip] Point à noter :
>
> Le Marketplace GitHub Actions est très riche. Pour la plupart des besoins courants (checkout, setup-node, setup-python, setup-java...), une action officielle existe déjà. Cherchez avant de tout réécrire.

