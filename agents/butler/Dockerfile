FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Copy shared utilities
COPY agents/shared /app/shared

# Copy butler code
COPY agents/butler/butler.py /app/butler/butler.py
COPY agents/butler/requirements.txt /app/butler/requirements.txt

# Install Python dependencies
RUN pip install --no-cache-dir -r /app/butler/requirements.txt

# Set environment variables
ENV PYTHONUNBUFFERED=1
ENV PYTHONPATH=/app

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python /app/butler/butler.py --health-check || exit 1

# Run butler in interactive mode by default
WORKDIR /app/butler
CMD ["python", "butler.py"]
