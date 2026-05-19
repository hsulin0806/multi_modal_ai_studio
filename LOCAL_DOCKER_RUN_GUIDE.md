# multi_modal_ai_studio 本機 Docker 啟動教學

## 目標
在本機使用 Docker 成功啟動並驗證 `multi_modal_ai_studio`。

## 1) 取得專案
```bash
cd /home/ubuntu/.openclaw/workspace
git clone https://github.com/NVIDIA-AI-IOT/multi_modal_ai_studio.git
cd multi_modal_ai_studio
```

## 2) 建立 `Dockerfile.local`
```bash
cat > Dockerfile.local <<'EOF'
FROM python:3.12-slim

ENV DEBIAN_FRONTEND=noninteractive \
    PIP_NO_CACHE_DIR=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

RUN apt-get update && apt-get install -y --no-install-recommends \
    openssl \
    ca-certificates \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . /app

RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip install --upgrade pip setuptools wheel && pip install -e .

EXPOSE 8092
CMD ["multi-modal-ai-studio", "--host", "0.0.0.0", "--port", "8092"]
EOF
```

## 3) 建立 `docker-compose.local.yml`
```bash
cat > docker-compose.local.yml <<'EOF'
services:
  mmas:
    image: mmas-local:py312
    container_name: mmas-local
    restart: unless-stopped
    ports:
      - "18092:8092"
    gpus: all
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - ./sessions:/app/sessions
      - ./videos:/app/videos
      - mmas_config:/root/.config/multi-modal-ai-studio

volumes:
  mmas_config:
EOF
```

## 4) Build + Run
```bash
docker build -f Dockerfile.local -t mmas-local:py312 .
docker compose -f docker-compose.local.yml up -d --force-recreate mmas
```

## 5) 驗證
```bash
docker ps --filter name=mmas-local
curl -k -I https://127.0.0.1:18092
curl -k https://127.0.0.1:18092/api/sessions
```

## 6) 使用入口
- `https://<主機IP>:18092`
- 首次會遇到自簽憑證，先手動接受即可。
