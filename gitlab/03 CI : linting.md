# Étape 3 — CI : linting et analyse statique

Le linting vérifie que votre code respecte les standards de l'équipe avant même de l'exécuter.

### Le principe des images Docker

Chaque job peut tourner dans une image Docker différente. C'est ce qui rend GitLab CI totalement agnostique du langage :

```yaml
stages:
  - lint

# PHP
lint_php:
  stage: lint
  image: php:8.2-cli
  script:
    - find . -name "*.php" -exec php -l {} \;
    - composer install --no-interaction
    - vendor/bin/phpstan analyse src/ --level=5

# Python
lint_python:
  stage: lint
  image: python:3.12
  script:
    - pip install pylint
    - pylint src/ --fail-under=8.0

# JavaScript / TypeScript
lint_js:
  stage: lint
  image: node:20
  script:
    - npm ci
    - npm run lint
```

- Ces trois jobs s'exécutent en parallèle car ils sont dans le même stage

- Si l'un échoue, le pipeline s'arrête et le développeur est notifié.

- Pour PHP, j'utilise [GrumPHP](https://github.com/phpro/grumphp), qui me permet de regrouper pleins d'outils en une commande : linter (phpcs et phpcs fixer), vérification du code (phpstan et phpcpd), tests (phpunit), sécurité (securitychecker_enlightn), ...  

---

>[!tip]  Point à noter :
>
> `npm ci` est préférable à `npm install` en CI : il est plus rapide, reproductible, et échoue si le package-lock.json est désynchronisé.