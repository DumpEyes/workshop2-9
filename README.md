# Workshop 2 — Analyse SAST avec SonarQube et Semgrep

**Module :** DevSecOps  
**Groupe :** 9  
**Membres :**
- Abdelouahab Es-Samadi
- Hatim Edderchoune
- Mouad Jmmali

**Projet analysé :** OWASP WebGoat  
**Dépôt GitHub :** https://github.com/DumpEyes/workshop2-9

---

## 1. Introduction

Ce workshop a pour objectif d'intégrer l'analyse statique de sécurité (SAST — Static Application Security Testing) dans un pipeline CI à l'aide de **SonarQube**, **Semgrep** et **GitHub Actions**.

Nous avons choisi **OWASP WebGoat**, une application Java basée sur Spring Boot et Maven, volontairement vulnérable et conçue à des fins pédagogiques.

L'objectif est de détecter des problèmes de sécurité dans le code source, de corriger un exemple de problème identifié et d'automatiser les analyses.

## 2. Environnement et outils

|            Outil            |                   Utilisation                   |
|-----------------------------|-------------------------------------------------|
| Git et GitHub               | Gestion du code source                          |
| GitHub Actions              | Automatisation du pipeline CI                   |
| Java 25 et Maven Wrapper    | Compilation et tests                            |
| SonarQube Community         | Analyse de la qualité et de la sécurité du code |
| Semgrep                     | Analyse SAST basée sur des règles               |
| Docker                      | Exécution locale du serveur SonarQube           |
| Windows Self-hosted Runner  | Exécution du build et de SonarQube              |
| Ubuntu GitHub-hosted Runner | Exécution de Semgrep                            |

## 3. Pipeline CI avec GitHub Actions

Le workflow est défini dans le fichier `.github/workflows/devsecops.yml`.

Il contient deux jobs :

**Job 1 — Build et SonarQube**

1. Récupération du code source.
2. Configuration de Java 25.
3. Compilation et tests avec Maven Wrapper (`clean verify`).
4. Analyse du code avec SonarQube.

Ce job s'exécute sur un runner Windows auto-hébergé. Le serveur SonarQube est exécuté localement avec Docker.

**Job 2 — Semgrep SAST**

1. Récupération du code source.
2. Installation de Python et Semgrep.
3. Analyse du projet avec les configurations `p/security-audit` et `p/java`.
4. Génération et archivage des résultats au format JSON.

Ce job s'exécute sur un runner Ubuntu fourni par GitHub.

**Résultat :** les deux jobs se sont terminés avec succès en 8 minutes et 41 secondes.

![Exécution réussie du pipeline GitHub Actions](screenshots/github-actions.png)

## 4. Résultats de SonarQube

L'analyse du projet avec SonarQube a produit les résultats suivants :

|       Indicateur       |       Résultat |
|------------------------|---------------:|
| Lignes de code         | Environ 17 000 |
| Security Issues        |             92 |
| Reliability Issues     |             47 |
| Maintainability Issues |             15 |
| Open Issues            |            141 |
| Security Issues Closed |              2 |
| Quality Gate           |         Passed |

![Tableau de bord SonarQube](screenshots/sonarqube-overview.png)

![Résultats de sécurité SonarQube](screenshots/sonarqube-security.png)

### Exemple de vulnérabilité détectée

SonarQube a identifié un problème de sécurité dans `JWTRefreshEndpoint.java`, à la ligne 73.

- **Catégorie :** Security
- **Sévérité :** Blocker
- **Problème :** utilisation d'une valeur sensible codée en dur dans le code source.

Cette pratique peut exposer des informations sensibles et doit être évitée dans une application de production.

![Exemple de vulnérabilité SonarQube](screenshots/sonarqube-issue.png)

## 5. Résultats de Semgrep

L'analyse Semgrep a été exécutée avec les configurations suivantes :

```bash
semgrep scan --config p/security-audit --config p/java .
```

**Résultats après modification du code :**

|     Indicateur      |               Résultat |
|---------------------|-----------------------:|
| Fichiers analysés   |                  1 134 |
| Règles exécutées    |                    117 |
| Findings détectés   |                     43 |
| Statut de l'analyse | Completed successfully |

Les résultats détaillés ont également été générés au format JSON et conservés comme artifact dans GitHub Actions.

Les findings ont été rapportés sans faire échouer automatiquement le pipeline CI.

![Résultats de Semgrep](screenshots/semgrep-results.png)

## 6. Exemple de correction de sécurité

Lors des analyses, un problème lié à l'absence de restriction explicite des méthodes HTTP a été identifié dans le fichier :

`src/main/java/org/owasp/webgoat/container/service/LabelDebugService.java`

Nous avons modifié les annotations `@RequestMapping` pour préciser les méthodes HTTP autorisées.

**Lecture de l'état du débogage :**

```java
method = RequestMethod.GET
```

**Modification de l'état du débogage :**

```java
method = RequestMethod.POST
```

Cette modification permet de restreindre explicitement les méthodes HTTP acceptées par ces deux endpoints.

Après modification, une nouvelle analyse Semgrep du fichier concerné n'a retourné aucun finding avec les configurations utilisées. Une nouvelle analyse SonarQube a également montré la fermeture de deux problèmes de sécurité.

Le nombre total de findings Semgrep du projet est passé de **45 à 43**.

Cette correction ne garantit pas à elle seule la sécurité complète des endpoints ; d'autres mécanismes, comme l'autorisation et la protection CSRF, peuvent être nécessaires.

![Correction HTTP GET](screenshots/fix-get.png)

![Correction HTTP POST](screenshots/fix-post.png)

## 7. Conclusion

Ce workshop nous a permis de mettre en pratique une démarche DevSecOps en intégrant deux outils SAST dans un pipeline GitHub Actions.

Nous avons réussi à :

- Compiler et tester le projet Java avec Maven.
- Exécuter une analyse SonarQube automatisée.
- Exécuter une analyse Semgrep automatisée.
- Identifier plusieurs problèmes de sécurité.
- Corriger un exemple de problème et relancer les analyses.
- Valider l'exécution du pipeline CI sur GitHub.

---

### Remarque

Les captures d'écran illustrant les différentes étapes du workshop (GitHub Actions, SonarQube, Semgrep et corrections apportées) sont disponibles dans le dossier **[`WebGoat\screenshots`](screenshots/)** à la racine du dépôt GitHub.

**Référence du projet original :** https://github.com/WebGoat/WebGoat

**** Workshop 2 — Groupe 9 — DevSecOps ***
