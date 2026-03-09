FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt gunicorn

COPY app.py .

# Utilisateur non-root (sécurité)
#RUN adduser --disabled-password --gecos '' appuser
#USER appuser

ENV DB_HOST=db
ENV DB_NAME=logs_db
ENV DB_USER=logs_user
ENV DB_PASSWORD=logs_password
ENV DB_PORT=5432

EXPOSE 5000

# gunicorn en prod — JAMAIS python app.py avec debug=True
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "app:app"]
