# -----------------------
# Base Stage
# -----------------------
FROM --platform=linux/amd64 node:18-slim AS base

# Install curl and tini for healthchecks and process management
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl tini && \
    rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /usr/local/app

# Install nodemon globally for local development (optional)
RUN npm install -g nodemon

# Copy package files and install dependencies
COPY package*.json ./
RUN npm ci && npm cache clean --force

# -----------------------
# Final Stage (production)
# -----------------------
FROM --platform=linux/amd64 node:18-slim AS final

# Install tini again in final image
RUN apt-get update && \
    apt-get install -y --no-install-recommends tini && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/app

# Copy node_modules from base stage
COPY --from=base /usr/local/app/node_modules ./node_modules

# Copy application source code
COPY . .

# Set environment variables
ENV PORT=80

# Expose port 80
EXPOSE 80

# Use tini as init and run Node.js server
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "server.js"]
