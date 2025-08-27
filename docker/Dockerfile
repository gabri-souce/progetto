# Dockerfile per l'app Python
FROM python:3.11-slim

WORKDIR /app

# Copia requirements
COPY requirements.txt .

# Installa dipendenze
RUN pip install --no-cache-dir -r requirements.txt

# Copia il codice
COPY app.py .

# Esponi porta
EXPOSE 5000

# Avvio app
CMD ["python", "app.py"]

