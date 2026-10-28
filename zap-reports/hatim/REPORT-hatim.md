\# Workshop 4 : OWASP ZAP sur WebGoat



Hatim, branche `workshop4-hatim`, 28/10/2026



\## 1. Ce que j'ai fait



J'ai scanné l'application WebGoat avec OWASP ZAP (version 2.17.0), le tout dans Docker. J'ai fait un baseline scan, donc un scan passif : ZAP regarde les réponses du serveur mais n'attaque pas l'application.



J'ai d'abord eu un problème avec Docker Desktop, dont le moteur ne démarrait pas. Je l'ai réparé en supprimant la distribution WSL `docker-desktop` pour qu'elle soit recréée. Ensuite, le port 8080 de mon PC était déjà pris par un autre programme, donc j'ai lancé WebGoat sur le port 8081 de ma machine. Dans le réseau Docker, ZAP le voit quand même sur le port 8080.



Les commandes :



```bat

docker network create zapnet

docker run -d --name webgoat --network zapnet -p 8081:8080 -p 9091:9090 webgoat/webgoat



mkdir zap-reports

docker run --rm --network zapnet -v "%cd%\\zap-reports:/zap/wrk:rw" ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://webgoat:8080/WebGoat -r baseline-report-hatim.html

```



Le rapport complet est dans `zap-reports/hatim/baseline-report-hatim.html`.



\## 2. Résultat du scan



Dans la console, ZAP affiche : FAIL 0, WARN 11, PASS 57. Dans le rapport HTML, on trouve 14 alertes. Ce n'est pas le même nombre parce que certaines règles regroupent plusieurs alertes (par exemple les trois headers Cross-Origin).



| Risque | Alertes |

|---|---|

| High | 0 |

| Medium | 3 |

| Low | 6 |

| Informational | 5 |



Il n'y a donc aucune alerte grave, mais trois alertes de niveau Medium.



\## 3. Les cookies



Le sujet demandait de regarder les cookies sans HttpOnly ou sans Secure. Dans mon scan, les deux contrôles sont passés (PASS), donc le cookie de session `JSESSIONID` a bien ces deux protections. Le seul souci est qu'il n'a pas l'attribut SameSite (risque Low). Sans SameSite, le navigateur peut envoyer le cookie lors d'une requête venant d'un autre site, ce qui aide les attaques CSRF. La solution proposée par ZAP est de mettre `SameSite=Lax` ou `Strict`.



\## 4. Les headers manquants



| Header | Risque | Pourquoi il est utile |

|---|---|---|

| X-Frame-Options (ou `frame-ancestors`) | Medium | Empêche d'afficher le site dans une iframe (clickjacking) |

| Content-Security-Policy | Medium | Limite les scripts autorisés, donc réduit l'impact d'un XSS |

| X-Content-Type-Options | Low | Empêche le navigateur de deviner le type d'un fichier |

| Permissions-Policy | Low | Restreint l'accès à la caméra, au micro, à la localisation |

| Cross-Origin-Embedder-Policy | Low | Isolation entre origines |

| Cross-Origin-Opener-Policy | Low | Isolation entre origines |

| Cross-Origin-Resource-Policy | Low | Protège contre des attaques de type Spectre |



\## 5. Les autres alertes



L'alerte Medium restante est l'absence de jetons anti-CSRF dans les formulaires de connexion et d'inscription (5 pages). Un site extérieur pourrait donc faire envoyer une requête à un utilisateur connecté sans qu'il le sache.



ZAP a aussi signalé en Informational un possible XSS sur la page `register.mvc` : les valeurs saisies dans le formulaire se retrouvent dans des attributs HTML. Le scan passif ne peut pas confirmer qu'il est exploitable, il faudrait un scan actif.



Les autres alertes Informational (détection de l'authentification, de la session, contenu cachable ou non) n'ont rien à corriger.



L'avertissement « spider error 404 » vient du fait que l'adresse `http://webgoat:8080/` n'existe pas, l'application est sous `/WebGoat`. Ça ne pose pas de problème.



\## 6. Risques et corrections



Je corrigerais d'abord les trois alertes Medium :



\- \*\*CSRF\*\* : ajouter un jeton unique dans chaque formulaire et le vérifier côté serveur.

\- \*\*CSP\*\* : définir un header `Content-Security-Policy` strict.

\- \*\*Clickjacking\*\* : ajouter `X-Frame-Options: DENY`.



Ensuite, les alertes Low : `SameSite=Lax` sur le cookie, `X-Content-Type-Options: nosniff`, et les headers Permissions-Policy, COEP, COOP et CORP. Ce sont surtout des réglages de configuration du serveur, ils ne demandent pas de changer le code de l'application.



\## 7. Ce que le scan passif ne montre pas



Le baseline scan ne détecte pas les vraies injections. Pour trouver des XSS ou des injections SQL, il faut un scan actif (`zap-full-scan.py`), qui envoie des requêtes d'attaque. Je ne l'ai pas fait dans ce TP, parce qu'il est long et qu'il ne faut le lancer que sur une instance de test que l'on contrôle, jamais sur un site réel sans autorisation.



\## 8. DAST dans un pipeline CI/CD



Voici comment j'intégrerais le DAST dans un pipeline :



\- Lancer un baseline scan à chaque Pull Request, parce qu'il est rapide et sans risque.

\- Lancer le scan actif plus rarement (la nuit, ou avant une mise en production) et seulement sur un environnement de test ou de pré-production.

\- Faire échouer le build si une alerte High apparaît, et laisser les alertes moins graves dans le rapport pour qu'on les traite ensuite.

\- Garder le rapport comme artefact du pipeline pour comparer d'une version à l'autre.

\- Prévoir un fichier de règles pour ignorer les faux positifs déjà analysés.

\- Configurer l'authentification de ZAP, sinon il ne voit pas les pages protégées.

\- Utiliser des données de test et avoir l'autorisation de scanner la cible.



Exemple avec GitHub Actions (à adapter, l'URL est un exemple et la version de l'action est à vérifier) :



```yaml

name: DAST baseline

on: \[pull\_request]

jobs:

&#x20; zap:

&#x20;   runs-on: ubuntu-latest

&#x20;   steps:

&#x20;     - name: ZAP Baseline Scan

&#x20;       uses: zaproxy/action-baseline@v0.14.0

&#x20;       with:

&#x20;         target: 'https://staging.example.com'

&#x20;         fail\_action: true

```



\## 9. Conclusion



Le scan montre que WebGoat n'a pas de faille grave visible en passif, mais qu'il lui manque plusieurs protections de base : jetons CSRF, CSP, protection contre le clickjacking, et l'attribut SameSite sur le cookie. Ces points sont faciles à corriger. Pour aller plus loin, il faudrait faire le scan actif. Avec un pipeline CI/CD, ce genre de contrôle se ferait automatiquement à chaque modification du code.

