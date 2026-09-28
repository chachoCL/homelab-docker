FROM python:3.11-slim

WORKDIR /app

# open-banking-chile runs locally through Node + Chromium. Xvfb supports banks
# that require a visible browser when OPEN_BANKING_XVFB=true.
RUN apt-get update && apt-get install -y --no-install-recommends \
    chromium \
    nodejs \
    npm \
    xauth \
    xvfb \
    && rm -rf /var/lib/apt/lists/*
ENV CHROME_PATH=/usr/bin/chromium

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . ./backend
RUN cd backend/open-banking-chile && npm ci && npm run build

EXPOSE 8000
CMD ["uvicorn", "backend.main:app", "--host", "0.0.0.0", "--port", "8000"]
