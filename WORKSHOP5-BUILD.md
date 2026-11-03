# Workshop 5 — Maven Build & Docker Build

## 1. Compilation Maven

Commande exécutée :

```powershell
.\mvnw.cmd -B clean verify
```

Résultat : **BUILD SUCCESS**

## 2. Construction de l'image Docker

Commande exécutée :

```powershell
docker build -t webgoat:workshop5 .
```

Résultat : image Docker créée avec succès.

Image : `webgoat:workshop5`

## 3. Vérification de l'image

```powershell
docker images webgoat
```

L'image est présente localement.

## 4. Test du conteneur

```powershell
docker run -d --name webgoat-w5 -p 127.0.0.1:8081:8080 webgoat:workshop5
```

Application accessible à l'adresse :

`http://localhost:8081/WebGoat`

Résultat : page de connexion WebGoat affichée avec succès.

## 5. Conclusion

La compilation Maven, la construction de l'image Docker et le test du conteneur ont été réalisés avec succès. L'image est prête pour une analyse de sécurité avec Trivy.