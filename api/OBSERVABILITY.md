# Mise en place de l'observabilite : Prometheus + Grafana

## Objectif

Ce document explique comment nous avons integre une stack d'observabilite dans le projet microservices `doodlestudent`.
L'objectif est de superviser le service backend avec Prometheus et de visualiser les donnees de monitoring avec Grafana.

## Perimetre

Composants implementes :

- Prometheus pour collecter les metriques
- Grafana pour visualiser les metriques

Fichiers modifies :

- `api/docker-compose.yaml`
- `api/prometheus.yml`

## Prerequis

- Docker et Docker Compose installes
- Backend executable avec Quarkus (`./mvnw compile quarkus:dev`)

## Etapes d'implementation

### 1) Ajouter la configuration Prometheus

Creer `api/prometheus.yml` :

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "quarkus-api"
    metrics_path: /q/metrics
    static_configs:
      - targets: ["host.docker.internal:8080"]
```

Cette configuration interroge l'endpoint de metriques Quarkus toutes les 5 secondes.

### 2) Ajouter les services Prometheus et Grafana

Dans `api/docker-compose.yaml`, ajouter :

- le service `prometheus` expose sur le port `9090`
- le service `grafana` expose sur le port `3000`
- le montage du fichier `prometheus.yml`

### 3) Demarrer les services

Depuis la racine du repository :

```bash
docker-compose -f "api/docker-compose.yaml" down
docker-compose -f "api/docker-compose.yaml" up -d
```

Lancer le backend (terminal separe) :

```bash
cd api
./mvnw compile quarkus:dev
```

Lancer le frontend si necessaire (terminal separe) :

```bash
cd front
npm install
npm start
```

## Validation

### Sante des targets Prometheus

- URL : `http://localhost:9090`
- Verifier `Status > Targets`
- Resultat attendu : la target `quarkus-api` est en etat `UP`

### Source de donnees Grafana

- URL : `http://localhost:3000`
- Identifiants par defaut : `admin / admin`
- Ajouter une datasource Prometheus avec l'URL : `http://prometheus:9090`
- Resultat attendu : `Successfully queried the Prometheus API`

### Verification d'une requete PromQL

La requete ci-dessous renvoie des donnees valides :

```promql
up{job="quarkus-api"}
```

La valeur attendue est `1`, ce qui signifie que Prometheus arrive bien a collecter les metriques du backend.

## Difficultes rencontrees et solutions (retour d'experience)

### Difficulte 1 : certaines metriques JVM ne renvoyaient pas de donnees

- Exemple : `base_cpu_system_load_average` renvoyait un resultat vide.
- Cause : la disponibilite des metriques depend de l'environnement d'execution et du jeu de metriques exposees.
- Solution : utiliser une metrique fiable pour valider l'integration :
  - `up{job="quarkus-api"}`

### Difficulte 2 : confusion sur le chemin d'execution de Docker Compose

- Lancer Compose depuis le mauvais dossier ne trouvait pas le fichier de configuration.
- Solution : executer Compose avec le chemin explicite depuis la racine du repository :
  - `docker-compose -f "api/docker-compose.yaml" ...`

## Resultat

Nous avons integre avec succes une premiere stack d'observabilite dans un projet microservices :

- Prometheus collecte les metriques du backend
- Grafana est connecte a Prometheus
- La chaine de monitoring est validee de bout en bout

## Ameliorations possibles

- Ajouter des dashboards Grafana preconfigures (provisioning JSON)
- Ajouter des regles d'alerte dans Prometheus
- Ajouter davantage de metriques applicatives et de labels
