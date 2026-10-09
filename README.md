\# Mon pipeline CI/CD avec Django, Docker et Traefik



Application web Django déployée automatiquement sur une VM Oracle Cloud (offre gratuite), avec HTTPS automatique et déploiement continu via GitLab CI/CD.



\*\*Démo en ligne :\*\* https://hussain-app.duckdns.org



\## Ce que fait ce projet



Chaque fois que je pousse du code sur GitLab, un robot (le "runner") installé sur mon propre serveur teste automatiquement le code, reconstruit l'image Docker, et redémarre l'application, sans que j'aie besoin de me connecter au serveur manuellement.



\## Stack technique



\- \*\*Django\*\* — framework web Python pour l'application

\- \*\*Docker / Docker Compose\*\* — conteneurisation de l'application

\- \*\*Traefik\*\* — reverse proxy, routage, et certificats HTTPS automatiques (Let's Encrypt)

\- \*\*GitLab CI/CD\*\* — pipeline d'intégration et déploiement continus

\- \*\*Runner auto-hébergé\*\* — le pipeline s'exécute sur mon propre serveur, pas sur les machines partagées de GitLab

\- \*\*Oracle Cloud (Always Free)\*\* — VM ARM (Ampere A1), 1 OCPU / 6 Go RAM, coût : 0€/mois

\- \*\*DuckDNS\*\* — nom de domaine gratuit



\## Comment ça marche



\### 1. Le serveur (Oracle Cloud)

Une VM Ubuntu tourne en continu sur l'offre gratuite d'Oracle. Deux pare-feux protègent le serveur : celui d'Oracle (au niveau du réseau) et `iptables` (sur la VM elle-même), tous deux ouverts uniquement sur les ports 22 (SSH), 80 et 443 (web).



\### 2. Traefik, la porte d'entrée

Traefik écoute sur les ports 80/443 et redirige chaque visiteur vers le bon conteneur. Il obtient et renouvelle automatiquement les certificats HTTPS via Let's Encrypt, je n'ai jamais eu à gérer un certificat manuellement.



\### 3. L'application Django

Le code tourne dans un conteneur Docker. Traefik sait comment l'atteindre grâce à des \*labels\* posés sur le conteneur (pas de configuration Traefik à modifier à chaque déploiement).



\### 4. Le pipeline CI/CD

Un fichier `.gitlab-ci.yml` définit deux étapes :

\- \*\*test\*\* : vérifie que l'image Docker se construit correctement

\- \*\*deploy\*\* : reconstruit et redémarre le conteneur, uniquement si les tests passent, et uniquement sur la branche `main`



Le runner qui exécute ce pipeline est installé directement sur ma VM. Il ne fait qu'interroger GitLab ("as-tu du travail pour moi ?"), donc aucun port supplémentaire n'a besoin d'être ouvert pour que ça fonctionne.



\### 5. Les secrets

La clé secrète Django (`SECRET\_KEY`) n'est jamais stockée dans le code. Elle est gardée dans les variables CI/CD de GitLab (masquées) et injectée dans un fichier `.env` au moment du déploiement.



\## Pipeline en action



!\[Pipeline réussi](screenshot-pipeline.png)



\*(Remplacer par une vraie capture d'écran d'un pipeline vert)\*



\## Pourquoi ce projet



Je voulais comprendre, de bout en bout, comment une application passe du code sur mon PC à un site en ligne, sans intervention manuelle, exactement comme en entreprise : serveur, conteneurisation, routage HTTPS, et intégration/déploiement continus. Chaque brique a été montée et configurée à la main, pas via un tutoriel "clé en main".



\## Prochaines étapes



Je travaille actuellement sur le déploiement de cette même application dans un cluster \*\*k3s\*\* (version légère de Kubernetes) sur la même VM, avec le Traefik existant comme contrôleur d'entrée (\*ingress\*) pour le cluster, afin d'apprendre l'orchestration de conteneurs en plus du simple Docker Compose.

