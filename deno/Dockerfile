FROM lscr.io/linuxserver/qbittorrent:latest

ENV WEBUI_PORT=8000
ENV TORRENTING_PORT=6881
ENV FILEBROWSER_PORT=8080
ENV FILEBROWSER_ROOT=/app/downloads
ENV FB_USERNAME=admin
ENV FB_PASSWORD=oga123456789

## Install dependencies
RUN apk add --no-cache curl ca-certificates

# Download cloudflared (multi‑arch)
ARG TARGETARCH
RUN case "${TARGETARCH}" in \
        amd64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64" ;; \
        arm64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64" ;; \
        *)      echo "Unsupported arch: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    curl -L --retry 5 --fail -o /usr/local/bin/cloudflared "$CF_URL" && \
    chmod +x /usr/local/bin/cloudflared

# Download filebrowser (multi‑arch)
RUN case "${TARGETARCH}" in \
        amd64)  FB_URL="https://github.com/filebrowser/filebrowser/releases/latest/download/linux-amd64-filebrowser.tar.gz" ;; \
        arm64)  FB_URL="https://github.com/filebrowser/filebrowser/releases/latest/download/linux-arm64-filebrowser.tar.gz" ;; \
        *)      echo "Unsupported arch: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    curl -L --retry 5 --fail "$FB_URL" | tar xz -C /usr/local/bin/ filebrowser && \
    chmod +x /usr/local/bin/filebrowser

# Create directories
RUN mkdir -p ${FILEBROWSER_ROOT} /config/filebrowser

# ------------------------------------------------------------
# Fix permissions: cont-init script runs after user is created
# ------------------------------------------------------------
RUN mkdir -p /etc/cont-init.d && \
    cat > /etc/cont-init.d/99-fix-downloads-perms << 'EOF'
#!/usr/bin/with-contenv bash
# This runs after the abc user is created (with PUID/PGID)
if [ -d /app/downloads ]; then
    echo "Fixing ownership of /app/downloads to abc:abc"
    chown -R abc:abc /app/downloads
    chmod 755 /app/downloads
fi
EOF
RUN chmod +x /etc/cont-init.d/99-fix-downloads-perms

# Startup script
RUN cat > /custom-init.sh << 'EOF'
#!/bin/sh

DB_PATH="/config/filebrowser/filebrowser.db"

# Reset and configure FileBrowser
rm -f "$DB_PATH"
filebrowser config init --database="$DB_PATH"
filebrowser config set \
    --address=0.0.0.0 \
    --port=$FILEBROWSER_PORT \
    --root=$FILEBROWSER_ROOT \
    --database="$DB_PATH"
filebrowser users add \
    "$FB_USERNAME" "$FB_PASSWORD" \
    --database="$DB_PATH" \
    --perm.admin

echo "FileBrowser credentials: $FB_USERNAME / $FB_PASSWORD"
echo "FileBrowser serving: $FILEBROWSER_ROOT"

# Start FileBrowser in background
/usr/local/bin/filebrowser --database="$DB_PATH" &
FB_PID=$!

# Wait for services to bind
sleep 2

# Start Cloudflare tunnel for qBittorrent (port 8000)
echo "Starting tunnel for qBittorrent (port $WEBUI_PORT)..."
/usr/local/bin/cloudflared tunnel --url http://localhost:$WEBUI_PORT --no-autoupdate &
CF_QBT_PID=$!

# Start Cloudflare tunnel for FileBrowser (port 8080)
echo "Starting tunnel for FileBrowser (port $FILEBROWSER_PORT)..."
/usr/local/bin/cloudflared tunnel --url http://localhost:$FILEBROWSER_PORT --no-autoupdate &
CF_FB_PID=$!

echo "All services started. qBittorrent tunnel PID: $CF_QBT_PID, FileBrowser tunnel PID: $CF_FB_PID"

# Run the original s6 init (starts qBittorrent)
exec /init
EOF

RUN chmod +x /custom-init.sh

EXPOSE 8000 6881 6881/udp 8080
ENTRYPOINT ["/custom-init.sh"]
