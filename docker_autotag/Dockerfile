# 1. FIX: Use "Bookworm" (Debian 12) instead of "Buster" (Debian 10)
# This fixes many OS-level vulnerabilities found in older images.
FROM python:3.10-slim-bookworm

# Prevent Python from writing pyc files and buffering stdout (better for logging)
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    AWS_REGION="us-east-1"

WORKDIR /app

# 2. SECURITY: Create a non-root user
# Running as root is a major security risk. We create 'appuser'.
RUN groupadd -r appuser && useradd -r -g appuser appuser

# 3. PERFORMANCE: Install dependencies *before* copying the app
# This utilizes Docker Layer Caching. If you change your code but not your
# requirements, Docker skips this slow step on rebuilds.
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# 4. COPY: Now copy the rest of the source code
COPY . .

# 5. PERMISSIONS: Create directories and grant permissions to the non-root user
# Since we aren't root anymore, we must explicitly own the directories.
RUN mkdir -p /output/AutotagPDF \
    /output/ExtractTextInfoFromPDF \
    /output/zipfile/images && \
    chown -R appuser:appuser /app /output

# Switch to the non-root user for runtime
USER appuser

# Optional: Healthcheck ensures the container is actually responsive
# HEALTHCHECK CMD python -c "import requests; requests.get('http://localhost:8080') || exit 1"

CMD ["python", "./autotag.py"]
