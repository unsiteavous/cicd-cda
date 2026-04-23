# Étape 5 — CI : sécurité SAST/SCA

GitLab intègre nativement des outils de sécurité, activables en trois lignes.

## Les templates GitLab Security

```yaml
include:
- template: Security/SAST.gitlab-ci.yml           # analyse du code source
- template: Security/Dependency-Scanning.gitlab-ci.yml  # dépendances vulnérables
- template: Security/Secret-Detection.gitlab-ci.yml     # secrets commités par erreur
```
- SAST (Static Application Security Testing) : analyse le code à la recherche de failles connues (injections SQL, XSS, etc.)

- Dependency Scanning : vérifie que vos librairies n'ont pas de CVE connues

- Secret Detection : détecte les mots de passe, clés API, tokens commités par erreur

## Voir les résultats

- Allez dans Security > Vulnerability Report après l'exécution du pipeline

- GitLab classe les vulnérabilités par niveau de criticité (Critical, High, Medium, Low)

>[!tip] Point à noter :
>
> Faites le test en direct : commitez un faux mot de passe en dur dans un fichier (password = "super_secret_123"), poussez, et regardez Secret Detection le détecter immédiatement. C'est l'argument le plus efficace pour convaincre une équipe de l'utilité de cet outil.