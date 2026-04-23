# Étape 2 — CI : hooks Git et vérifications locales

La CI ne commence pas dans GitHub — elle commence sur votre machine, avant même le push.

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

    Testez avec un bon commit : `git commit -m "feat(auth): ajout de la connexion SSO"` → il passe

## Husky (pour les projets JS/TS)

`.git/hooks/` n'est pas versionné, ce qui pose problème en équipe. (Husky)[https://typicode.github.io/husky/] résout ça en versionnant vos hooks :

```bash
npm install --save-dev husky
npx husky init
```

Les hooks sont alors dans `.husky/` et commités avec le projet — tous les développeurs de l'équipe les ont automatiquement.

## Vérification du nom de branche dans GitHub Actions

```yml
jobs:
check_branch_name:
runs-on: ubuntu-latest
steps:
  - name: Vérifier le nom de branche
    run: |
      if ! echo "${{ github.ref_name }}" | grep -qE "^(main|develop|feature/.+|fix/.+|release/.+)$"; then
        echo "❌ Nom de branche invalide : ${{ github.ref_name }}"
        echo "👉 Format attendu : feature/ma-fonctionnalite, fix/mon-bug, ..."
        exit 1
      fi
```
J'ai fait une série d'articles à ce sujet qui sont plus complets sur mon site : https://unsiteavous.fr/astuces/ et ciblez husky.

>[!Tip] Point à noter :
>
> Chez GitLab, on utilisait `$CI_COMMIT_BRANCH`. Chez GitHub, la syntaxe est `${{ github.ref_name }}`. C'est le contexte GitHub — l'équivalent des variables prédéfinies GitLab. Consultez la (documentation officielle)[https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs].