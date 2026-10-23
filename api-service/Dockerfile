# ==========================================
# Stage 1: Build Stage
# ==========================================
FROM node:22.23.4-bookworm-slim AS builder

WORKDIR /app

# Install system dependencies required for Playwright
RUN apt-get update && apt-get install -y \
    wget \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Install Playwright Chromium browser and system dependencies
RUN npx playwright install chromium --with-deps

# ==========================================
# Stage 2: Production Stage
# ==========================================
FROM node:22.23.4-bookworm-slim

WORKDIR /app

# Install runtime dependencies for Chromium
RUN apt-get update && apt-get install -y \
    # Chromium dependencies
    libnss3 \
    libnspr4 \
    libatk1.0-0 \
    libatk-bridge2.0-0 \
    libcups2 \
    libdrm2 \
    libdbus-1-3 \
    libxkbcommon0 \
    libxcomposite1 \
    libxdamage1 \
    libxfixes3 \
    libxrandr2 \
    libgbm1 \
    libpango-1.0-0 \
    libcairo2 \
    libasound2 \
    libatspi2.0-0 \
    # Fonts
    fonts-liberation \
    fonts-noto-color-emoji \
    # Clean up
    && rm -rf /var/lib/apt/lists/*

# Copy node_modules from builder
COPY --from=builder /app/node_modules ./node_modules

# Copy Playwright browsers to a shared location
COPY --from=builder /root/.cache/ms-playwright /ms-playwright

# Copy application files
COPY server.js .
COPY worker.html .
COPY docs.html .
COPY wechat_files ./wechat_files

# Create non-root user for security
RUN groupadd -r playwright && \
    useradd -r -g playwright -G audio,video playwright && \
    mkdir -p /home/playwright && \
    chown -R playwright:playwright /home/playwright && \
    chown -R playwright:playwright /app && \
    chown -R playwright:playwright /ms-playwright

# Set Playwright browsers path
ENV PLAYWRIGHT_BROWSERS_PATH=/ms-playwright

# Switch to non-root user
USER playwright

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => { process.exit(r.statusCode === 200 ? 0 : 1); }).on('error', () => process.exit(1));"

# Start server
CMD ["node", "server.js"]
