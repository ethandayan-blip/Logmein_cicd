
[LogMeIn-GNS3-ReadMe.pdf](https://github.com/user-attachments/files/25903400/LogMeIn-GNS3-ReadMe.pdf)

# LogMeIn — GNS3 Global view

![GNS3setup](https://github.com/user-attachments/assets/d37f901f-86e0-4d83-a8ce-934b350e836c)

# LogMeIn — CI/CD Pipeline Documentation

## Architecture Overview

```
Developer Push
      │
      ▼
GitHub Actions CI
      ├── Lint (Flake8)
      ├── Tests (pytest + PostgreSQL)
      └── Security Scan (Trivy)
              │
              ▼ (push sur main)
GitHub Actions CD
      ├── Build images Docker
      └── Push → Docker Hub
              │
              ▼ (deploy manuel)
Docker Swarm
      ├── Manager (hkw)
      ├── Worker 1
      └── Worker 2
```

---

## Stack Technique

| Composant | Technologie |
|-----------|-------------|
| Backend | Python 3.11 / Flask / Gunicorn |
| Frontend | Nginx / HTML-CSS-JS |
| Base de données | PostgreSQL 15 |
| Containerisation | Docker / Docker Compose |
| Orchestration | Docker Swarm |
| CI/CD | GitHub Actions |
| Registry | Docker Hub |
| Security Scan | Trivy |
| Lint | Flake8 |
| Tests | Pytest |

---

## Structure du Repository

```
logmein/
├── backend/
│   ├── app.py                  # API Flask
│   ├── requirements.txt        # Dépendances Python (inclut gunicorn)
│   ├── requirements.dev.txt    # Dépendances de test (pytest, pytest-flask)
│   ├── Dockerfile              # Image sécurisée Python 3.11.9
│   ├── run_tests.sh
│   └── tests/
│       └── test_api.py         # Tests des endpoints API
├── frontend/
│   ├── index.html
│   ├── style.css
│   ├── script.js
│   ├── nginx.conf              # Proxy vers le backend
│   └── Dockerfile              # Image nginx:alpine avec apk upgrade
├── docker-compose.yml          # Environnement local / développement
├── docker-stack.yml            # Déploiement Docker Swarm production
├── .trivyignore                # CVE sans fix documentées
└── .github/
    └── workflows/
        ├── ci.yml              # Pipeline CI
        └── cd.yml              # Pipeline CD
```

---

## CI Pipeline

**Fichier :** `.github/workflows/ci.yml`

**Déclenchement :** Push sur toutes les branches + PR vers main

### Jobs

#### 1. Lint — Flake8
- Analyse statique de `backend/app.py`
- Détecte les erreurs de syntaxe et de style
- Bloque la suite si erreur critique

#### 2. Tests — Pytest
- Démarre un container PostgreSQL comme service GitHub Actions
- Lance les tests d'intégration sur les 5 endpoints de l'API
- Variables d'environnement injectées pour pointer vers la DB de test

#### 3. Build + Scan — Trivy
- Build les images Docker backend et frontend
- Scan de vulnérabilités sur chaque image
- **Bloque le pipeline si CVE CRITICAL détectée**
- CVE sans fix disponible documentées dans `.trivyignore`

### Résultat attendu

```
lint      ✅ Flake8 — aucune erreur critique
test      ✅ 6/6 tests passés
build     ✅ Images buildées
trivy     ✅ 0 CVE CRITICAL (2 ignorées — will_not_fix)
```

---

## CD Pipeline

**Fichier :** `.github/workflows/cd.yml`

**Déclenchement :** Push sur `main` uniquement (après merge)

### Jobs

#### Build & Push — Docker Hub
- Login sur Docker Hub via secrets GitHub
- Build et push `harkayn/logmein-backend:latest` + tag SHA
- Build et push `harkayn/logmein-frontend:latest` + tag SHA

### Images sur Docker Hub

```
harkayn/logmein-backend:latest
harkayn/logmein-frontend:latest
```

---

## Infrastructure Docker Swarm

### Cluster

| Nœud | Rôle | Hostname |
|------|------|----------|
| Manager | Leader | hkw |
| Worker 1 | Worker | 7396f287eea9 |
| Worker 2 | Worker | 85f7f6ac1f45 |

### Services en production

```bash
docker service ls
# NAME               REPLICAS   IMAGE
# logmein_backend    2/2        harkayn/logmein-backend:latest
# logmein_db         1/1        postgres:15
# logmein_frontend   2/2        harkayn/logmein-frontend:latest
```

### Déploiement manuel

```bash
export DOCKER_USERNAME=harkayn
docker stack deploy -c docker-stack.yml --with-registry-auth logmein
```

### Mise à jour des services

```bash
docker service update --image harkayn/logmein-backend:latest logmein_backend
docker service update --image harkayn/logmein-frontend:latest logmein_frontend
```

### Scaling

```bash
# Monter à 3 replicas backend
docker service scale logmein_backend=3
```

---

## Mesures de Sécurité

| Mesure | Implémentation |
|--------|---------------|
| Pas de debug en prod | Gunicorn remplace `python app.py` |
| Credentials sécurisés | Docker Secrets pour `db_password` et `api_key` |
| Réseau chiffré | Overlay network avec `encrypted: "true"` |
| User non-root | `appuser` (UID 1001) dans le Dockerfile backend |
| CVE patchées | `apt upgrade` + `apk upgrade` dans les Dockerfiles |
| Scan automatique | Trivy bloque sur CVE CRITICAL à chaque push |
| Token registry | PAT Docker Hub scope Read/Write (pas mot de passe) |
| Secrets GitHub | `DOCKER_USERNAME`, `DOCKER_PASSWORD` jamais en clair |

### CVE documentées (.trivyignore)

| CVE | Package | Statut | Raison |
|-----|---------|--------|--------|
| CVE-2023-45853 | zlib1g | will_not_fix | Pas de patch Debian disponible |
| CVE-2025-7458 | libsqlite3 | affected | Pas de fix version disponible |

---

## Variables d'environnement

### Backend

| Variable | Description | Valeur par défaut |
|----------|-------------|-------------------|
| `DB_HOST` | Hôte PostgreSQL | `db` |
| `DB_NAME` | Nom de la base | `logs_db` |
| `DB_USER` | Utilisateur DB | `logs_user` |
| `DB_PASSWORD` | Mot de passe DB | Via Docker Secret |
| `DB_PORT` | Port PostgreSQL | `5432` |
| `FLASK_DEBUG` | Mode debug Flask | `false` |

### GitHub Actions Secrets

| Secret | Usage |
|--------|-------|
| `DOCKER_USERNAME` | Login Docker Hub |
| `DOCKER_PASSWORD` | Token Docker Hub (PAT) |

---

## Lancer en local

```bash
# Cloner le repo
git clone https://github.com/ethandayan-blip/Logmein_cicd
cd Logmein_cicd

# Démarrer les services
docker compose up -d

# Vérifier
docker compose ps

# Accès
# Frontend : http://localhost:3000
# Backend  : http://localhost:3000/api/health
```

---

## API Endpoints

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| GET | `/health` | Santé de l'API + DB |
| GET | `/logs` | Liste des logs (pagination) |
| POST | `/logs` | Ajouter un log |
| GET | `/stats` | Statistiques par niveau/service |
| DELETE | `/logs/clear` | Vider tous les logs |

---

## Auteurs

Projet réalisé dans le cadre du cursus **JEDHA Fullstack Cybersecurity**.

par team BR34CH
