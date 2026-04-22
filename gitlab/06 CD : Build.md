# Étape 6 — CD : build et artefacts

Le build transforme votre code source en quelque chose de déployable.

### Quand en a-t-on besoin ?
| Technologie |	Build nécessaire ? |
|--|--|
|PHP natif              |	❌ (sauf assets front)|
|Python	                | ❌ (sauf packaging)|
|Java / .NET	          | ✅ (compilation obligatoire)|
|React / Vue / Angular	| ✅ (npm run build)|
|Node.js (API)	        | ⚠️ Selon le projet|

### Le job de build

```text
build:
  stage: build
  script:
    - ./scripts/build.sh
  artifacts:
    paths:
      - dist/         # adaptez selon votre projet : target, build/, vendor/...
    expire_in: 1 hour
  only:
    - main
    - develop
```
Le fichier `scripts/build.sh` contient la commande propre à votre projet :

```bash
# Exemple Java
mvn package -DskipTests

# Exemple .NET
dotnet publish -c Release -o dist/

# Exemple React
npm ci && npm run build
```
### Les artefacts

- Les artefacts sont les fichiers produits par un job et transmis aux jobs suivants

- Sans artefacts déclarés, chaque job repart d'un environnement vierge

- expire_in: 1 hour évite d'encombrer le stockage GitLab

>[!tip] Point à noter :
>
> Ne recompilez jamais deux fois. Ce qui est buildé et testé en étape 4 doit être exactement ce qui est déployé en étape 7. Les artefacts garantissent cela.