# Étape 6 — CD : build et artefacts

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: ./scripts/build.sh

      - name: Sauvegarder les artefacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/           # adaptez : target/, build/, vendor/...
          retention-days: 1
```
Pour récupérer les artefacts dans un job suivant :

```yml
  deploy:
    needs: build          # ce job attend que build soit terminé
    runs-on: ubuntu-latest
    steps:
      - name: Récupérer les artefacts
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/
```
>[!Tip] Point à noter :
>
> Chez GitHub, les artefacts ne transitent pas automatiquement entre les jobs comme chez GitLab. Il faut explicitement uploader avec `upload-artifact` puis télécharger avec `download-artifact`. Et les jobs doivent être chaînés avec needs: pour s'exécuter dans le bon ordre.