# ============================================================
# frontend/Dockerfile — VERSION SÉCURISÉE
# Diff avec ORIGINAL_frontend_Dockerfile indiqué en commentaires
# ============================================================

# CHANGEMENT 1 : Image versionnée
# ORIGINAL : FROM nginx:alpine  ← tag flottant
FROM nginx:1.27-alpine

# CHANGEMENT 2 : Supprimer la config nginx par défaut
# ORIGINAL : non présent — la config default nginx reste active
RUN rm /etc/nginx/conf.d/default.conf

COPY index.html /usr/share/nginx/html/
COPY style.css /usr/share/nginx/html/
COPY script.js /usr/share/nginx/html/

# CHANGEMENT 3 : Activer notre nginx.conf personnalisé
# ORIGINAL : cette ligne était commentée (TODO)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# CHANGEMENT 4 : Utilisateur non-root pour nginx
# ORIGINAL : nginx tourne en root
RUN adduser --disabled-password --gecos "" --uid 1001 nginxuser \
    && chown -R nginxuser:nginxuser /usr/share/nginx/html \
    && chown -R nginxuser:nginxuser /var/cache/nginx \
    && touch /var/run/nginx.pid \
    && chown nginxuser:nginxuser /var/run/nginx.pid

USER nginxuser

EXPOSE 80
