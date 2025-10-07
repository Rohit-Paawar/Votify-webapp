# -----------------------
# Base Stage
# -----------------------
FROM --platform=linux/amd64 python:3.11-slim AS base

# Install curl (optional, for healthchecks) and other dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /usr/local/app

# Copy requirements.txt and install dependencies
COPY requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# -----------------------
# Final Stage (production)
# -----------------------
FROM --platform=linux/amd64 python:3.11-slim AS final

# Install curl again if needed
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /usr/local/app

# Copy dependencies from base stage
COPY --from=base /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=base /usr/local/bin /usr/local/bin

# Copy application code
COPY . .

# Expose port
EXPOSE 80

# Run Gunicorn with 4 workers
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]
