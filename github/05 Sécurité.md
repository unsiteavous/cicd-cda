# Étape 5 — CI : sécurité SAST/SCA

GitHub propose CodeQL, son moteur d'analyse de sécurité natif, activable en quelques lignes :

```yml
jobs:
  security:
    runs-on: ubuntu-latest
    permissions:
      security-events: write    # obligatoire pour publier les résultats
    steps:
      - uses: actions/checkout@v4

      - name: Initialiser CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript, python, java   # adaptez à votre projet

      - name: Analyser avec CodeQL
        uses: github/codeql-action/analyze@v3

      - name: Scanner les dépendances vulnérables
        uses: actions/dependency-review-action@v4

      - name: Détecter les secrets commités
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
```
-  `CodeQL` : analyse le code source à la recherche de failles (équivalent SAST de GitLab)

-  `dependency-review-action` : vérifie les dépendances vulnérables (équivalent Dependency Scanning)

-  `TruffleHog` : détecte les secrets commités par erreur (équivalent Secret Detection)

-  Les résultats apparaissent dans Security > Code scanning alerts

>[!Tip] Point à noter :
>
> Faites le test en direct : commitez un faux mot de passe en dur (password = "super_secret_123"), poussez, et regardez TruffleHog le détecter immédiatement.