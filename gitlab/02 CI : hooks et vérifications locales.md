# Étape 2 — CI : hooks Git et vérifications locales

La CI ne commence pas dans GitLab — elle commence sur votre machine, avant même le push.

## Les hooks Git natifs

Un hook Git est un script qui se déclenche automatiquement à certains moments du cycle Git. Ils se trouvent dans `.git/hooks/`.

Le hook `commit-msg` vérifie le format de votre message de commit avant qu'il soit enregistré :

```bash
# .git/hooks/commit-msg
#!/bin/sh
commit_msg=$(cat "$1")
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+"; then
  echo "❌ Format de commit invalide."
  echo "👉 Utilisez le format Conventional Commits : feat(scope): description"
  exit 1
fi
```
- Rendez-le exécutable : `chmod +x .git/hooks/commit-msg`

- Testez avec un mauvais commit : `git commit -m "modif truc"` → il sera bloqué

- Testez avec un bon commit : `git commit -m "feat(auth): ajout de la connexion SSO"` → il passe

## Husky (pour les projets JS/TS)

`.git/hooks/` n'est pas versionné, ce qui pose problème en équipe. Husky résout ça en versionnant vos hooks :

```bash
npm install --save-dev husky
npx husky init
```
Les hooks sont alors dans `.husky/` et commités avec le projet — tous les développeurs de l'équipe les ont automatiquement.
Vérification du nom de branche dans GitLab.

En complément des hooks locaux, on peut vérifier le nom de branche côté pipeline :

```text
check_branch_name:
stage: verify
script:
- |
  if ! echo "$CI_COMMIT_BRANCH" | grep -qE "^(main|develop|feature/.+|fix/.+|release/.+)$"; then
    echo "❌ Nom de branche invalide : $CI_COMMIT_BRANCH"
    echo "👉 Format attendu : feature/ma-fonctionnalite, fix/mon-bug, ..."
    exit 1
  fi
rules:
- if: '$CI_COMMIT_BRANCH != "main" && $CI_COMMIT_BRANCH != "develop"'
```

J'ai fait une série d'articles à ce sujet qui sont plus complets sur mon site : https://unsiteavous.fr/astuces/ et ciblez husky.

>[!tip]  Point à noter :
>
> `$CI_COMMIT_BRANCH` est une variable prédéfinie par GitLab. Il en existe des dizaines (`$CI_COMMIT_SHA`, `$CI_PROJECT_NAME`, `$CI_PIPELINE_URL`...). Consultez la documentation officielle.