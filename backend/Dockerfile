# ============================================================
# backend/Dockerfile — VERSION SÉCURISÉE
# Diff avec ORIGINAL_backend_Dockerfile indiqué en commentaires
# ============================================================

# CHANGEMENT 1 : Image versionnée (tag fixe, pas flottant)
# ORIGINAL : FROM python:3.11-slim  ← tag flottant, peut changer silencieusement
FROM python:3.11.9-slim

WORKDIR /app

# CHANGEMENT 2 : --no-install-recommends réduit la surface d'attaque
# ORIGINAL : apt-get install -y gcc libpq-dev
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY requirements.txt .

# CHANGEMENT 3 : --no-cache-dir évite de stocker le cache pip dans l'image
# ORIGINAL : RUN pip install -r requirements.txt
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

COPY app.py .

# CHANGEMENT 4 : SUPPRESSION des credentials hardcodés dans l'image
# ORIGINAL :
#   ENV DB_HOST=db
#   ENV DB_NAME=logs_db
#   ENV DB_USER=logs_user
#   ENV DB_PASSWORD=logs_password   ← visible dans docker inspect / docker history
#   ENV DB_PORT=5432
#
# Ces valeurs seront injectées par docker-compose ou Docker Swarm secrets
# On garde seulement les non-sensibles sans valeur par défaut
ENV DB_HOST="" \
    DB_NAME="" \
    DB_USER="" \
    DB_PASSWORD="" \
    DB_PORT="5432" \
    FLASK_DEBUG="false" \
    API_KEY=""

EXPOSE 5000

# CHANGEMENT 5 : Créer un utilisateur non-root
# ORIGINAL : aucun USER défini → tourne en root (UID 0)
RUN adduser --disabled-password --gecos "" --uid 1001 appuser \
    && chown -R appuser:appuser /app

USER appuser

# CHANGEMENT 6 : Gunicorn (serveur de production) à la place du serveur dev Flask
# ORIGINAL : CMD ["python", "app.py"]  ← serveur Werkzeug de développement
# Gunicorn = WSGI de production, multi-workers, stable, sans debugger exposé
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "--timeout", "60", "app:app"]
