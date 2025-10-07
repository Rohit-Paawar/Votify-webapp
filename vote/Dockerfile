# Base image for ARM64
FROM --platform=linux/arm64 python:3.11-slim

# Add curl (optional) and clean up
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Set working directory inside the container
WORKDIR /app

# Copy requirements first and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy all app code and folders into the container
COPY . .

# Expose port for the app (Flask runs on 80 in your original Dockerfile)
EXPOSE 80

# Set entrypoint using gunicorn (production-ready WSGI server)
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]
