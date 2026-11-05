# on copie une image existante

FROM python:3.12-slim

# on choisi son répertoire de travail

WORKDIR /app

# on copie les fichiers de dépendances

COPY requirements.txt .

# on installe les dépendances

RUN pip install --no-cache-dir -r requirements.txt

# on copie le code source de l'application
COPY src/ .

# on expose le port sur lequel l'application va tourner

EXPOSE 8000

# on lance l'application

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]