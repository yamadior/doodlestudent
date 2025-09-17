# README_BIS

Mode d'emploi pour construire et lancer le projet via Docker (front + back + services annexes)

Prérequis
- Docker Desktop (démon en marche)
- (optionnel) Node.js/npm et Maven si tu veux builder localement sans Docker

1) Lancer tous les services via Docker Compose (méthode recommandée)

Ouvre un terminal dans le dossier `api` :

cd C:\Work\University\ALA\doodlestudent\api

docker-compose up --build -d

Vérifier l'état :

docker-compose ps

docker ps -a

Voir les logs :

docker-compose logs -f back

docker-compose logs -f front

Points d'accès
- Front : http://localhost:4200
- Back : http://localhost:8081  (mapping hôte -> container : 8081:8080)
- Etherpad : http://localhost:9001
- MySQL : 3306 (hôte)
- SMTP test : 2525 (hôte)

Monitoring - URLs importantes
- Prometheus (UI & targets) : http://localhost:9090
  - Targets : http://localhost:9090/targets
- Grafana (UI) : http://localhost:3000  (login par défaut : admin / admin)
- Application (Quarkus) - métriques : http://localhost:8081/q/metrics
- node-exporter (métriques système) : http://localhost:9100/metrics
- cAdvisor (containers) : http://localhost:8082/  (UI), métriques : http://localhost:8082/metrics
- mysqld-exporter (métriques MySQL) : accessible depuis le réseau Docker à http://mysqld-exporter:9104/metrics
  - Si tu veux y accéder depuis l'hôte, ajoute un mapping de port dans docker-compose (ex. `- "9104:9104"` pour le service mysqld-exporter).

Notes :
- node-exporter et cAdvisor donnent les meilleures métriques sur Linux / WSL2. Sur Windows natif, utilise windows_exporter si nécessaire.
- Grafana est provisionné pour utiliser Prometheus comme datasource ; les dashboards sont dans `api/grafana/dashboards`.

Arrêter et supprimer les conteneurs :

docker-compose down

Si tu veux supprimer aussi les volumes (reset BDD) :

docker-compose down -v

2) Rebuilder uniquement un service (exemples)

Reconstruire le front :

docker-compose build --no-cache front --progress=plain

Reconstruire le back :

docker-compose build --no-cache back --progress=plain

3) Builder et lancer manuellement (sans docker-compose)

Front (local / debug)
cd C:\Work\University\ALA\doodlestudent\front
npm install --legacy-peer-deps
npm run build
# construire l'image et lancer
docker build -t doodle-front .
docker run -p 4200:80 doodle-front

Back (JVM)
cd C:\Work\University\ALA\doodlestudent\api
# sur Windows PowerShell utilise mvnw.cmd
./mvnw package -DskipTests
# ou sous PowerShell : .\mvnw.cmd -DskipTests
# construire et lancer l'image
docker build -f src/main/docker/Dockerfile.jvm -t api-back .
docker run -p 8081:8080 api-back

4) Résolution de conflits de ports
Si un port hôte est déjà occupé (ex. 8080) :
- soit libérer le port (arrêter le processus qui l'utilise),
- soit modifier le mapping dans `api/docker-compose.yaml` (ex : `8081:8080`) et relancer `docker-compose up --build`.

5) Conseils rapides
- Si le build Maven télécharge beaucoup de dépendances, laisse-le terminer (peut prendre plusieurs minutes).
- Augmente la mémoire CPU/RAM de Docker Desktop si build échoue pour cause de ressources (recommandé 8–12GB RAM pour builds lourds).
- Pour debug rapide : `docker-compose logs -f back` et `docker-compose logs -f front`.

Si tu veux, j'adapte les ports ou j'intègre un healthcheck/wait-for script pour que le back attende la BDD. Indique ce que tu veux automatiser ensuite.
