# Monitoring Setup – Prometheus & Grafana

Ce document récapitule toute la mise en place du monitoring du projet avec **Prometheus** et **Grafana**.

L'objectif est de collecter les métriques du backend et de les visualiser via des dashboards.

---

# 1. Architecture du monitoring

Containers utilisés :

- backend (Flask API)
- db (PostgreSQL)
- frontend (Nginx)
- prometheus (collecte de métriques)
- grafana (visualisation)

Flux de monitoring :

Backend expose des métriques :

http://backend:5000/metrics

Prometheus les collecte :

Prometheus → scrape → backend

Grafana interroge Prometheus :

Grafana → query → Prometheus

Architecture complète :

Flask Backend → Prometheus → Grafana

---

# 2. Configuration Docker Compose

Dans `docker-compose.yml` :

```yaml
prometheus:
  image: prom/prometheus:latest
  ports:
    - "9090:9090"
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml
    - prometheus_data:/prometheus
  command:
    - '--config.file=/etc/prometheus/prometheus.yml'
    - '--storage.tsdb.path=/prometheus'
    - "--storage.tsdb.retention.time=15d"
  networks:
    - monitor-net

grafana:
  image: grafana/grafana:latest
  ports:
    - "3100:3000"
  volumes:
    - grafana_data:/var/lib/grafana
  networks:
    - monitor-net
  depends_on:
    - prometheus
```

Important :

Prometheus et Grafana doivent être sur le **même réseau Docker**.

```yaml
networks:
  monitor-net:
```

---

# 3. Configuration Prometheus

Fichier :

```
prometheus.yml
```

Configuration minimale :

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'backend'

    static_configs:
      - targets: ['backend:5000']
```

Prometheus scrape donc :

```
http://backend:5000/metrics
```

---

# 4. Exposition des métriques Flask

Installer la librairie :

```
pip install prometheus-client
```

Exemple minimal :

```python
from flask import Flask, Response
from prometheus_client import generate_latest

app = Flask(__name__)

@app.route("/metrics")
def metrics():
    return Response(generate_latest(), mimetype="text/plain")
```

Tester l'endpoint :

```
curl http://localhost:5000/metrics
```

Exemples de métriques :

```
process_cpu_seconds_total
process_resident_memory_bytes
python_gc_objects_collected_total
```

---

# 5. Lancer les containers

Depuis la racine du projet :

```
docker-compose up -d
```

Vérifier :

```
docker ps
```

Containers attendus :

```
backend
frontend
db
prometheus
grafana
```

---

# 6. Vérifier Prometheus

Interface Prometheus :

```
http://localhost:9090
```

Menu :

```
Status → Targets
```

Le backend doit être :

```
UP
```

Si c'est `DOWN`, vérifier :

```
http://backend:5000/metrics
```

---

# 7. Accéder à Grafana

Interface Grafana :

```
http://localhost:3100
```

Login par défaut :

```
username: admin
password: admin
```

---

# 8. Ajouter Prometheus comme Data Source

Dans Grafana :

Menu :

```
Connections → Data Sources
```

Ajouter :

```
Prometheus
```

URL :

```
http://prometheus:9090
```

Cliquer :

```
Save & Test
```

---

# 9. Création d'un dashboard Grafana

Menu :

```
Dashboards → New → New Dashboard
```

Puis :

```
Add Visualization
```

Choisir la data source :

```
Prometheus
```

---

# 10. Exemples de requêtes Prometheus

CPU usage :

```
rate(process_cpu_seconds_total[1m])
```

Memory usage :

```
process_resident_memory_bytes
```

Virtual memory :

```
process_virtual_memory_bytes
```

Open file descriptors :

```
process_open_fds
```

---

# 11. Importer un dashboard Grafana

Méthode recommandée.

Menu :

```
Dashboards → Import
```

Importer :

```
flask_dashboard.json
```

Choisir data source :

```
Prometheus
```

Puis :

```
Import
```

---

# 12. Dashboard obtenu

Le dashboard contient les panels suivants :

- CPU usage
- Memory usage
- Virtual memory
- Open file descriptors

Ces métriques proviennent du **process Python exposé par prometheus_client**.

---

