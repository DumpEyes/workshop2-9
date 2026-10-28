# Workshop 4 — Préparation de l'environnement DAST

## Objectif

Préparer l'application OWASP WebGoat pour effectuer des tests de sécurité dynamiques avec OWASP ZAP.

## Environnement

- Application : OWASP WebGoat
- Langage : Java
- Framework : Spring Boot
- Outil de build : Maven
- Système d'exploitation : Windows
- Outil de scan prévu : OWASP ZAP (Docker)

## Démarrage de WebGoat

Ouvrir un terminal PowerShell à la racine du projet WebGoat, puis exécuter :

```powershell
.\mvnw.cmd spring-boot:run
```

Cette commande utilise Maven Wrapper pour démarrer l'application Spring Boot.

## Vérification

Le démarrage de l'application a été effectué avec succès dans l'environnement local.

## Prochaines étapes

- Exécuter un Baseline Scan avec OWASP ZAP.
- Générer et analyser le rapport de sécurité.
- Étudier l'intégration du DAST dans un pipeline CI/CD.