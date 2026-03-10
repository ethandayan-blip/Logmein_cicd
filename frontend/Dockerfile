
FROM nginx:alpine

# Mise à jour des paquets pour patcher les CVE (zlib CVE-2026-22184)
RUN apk upgrade --no-cache

COPY index.html /usr/share/nginx/html/
COPY style.css /usr/share/nginx/html/
COPY script.js /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
