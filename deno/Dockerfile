FROM lscr.io/linuxserver/qbittorrent:latest

ENV WEBUI_PORT=8000
ENV TORRENTING_PORT=6881
ENV FILEBROWSER_PORT=8080
ENV FILEBROWSER_ROOT=/app/downloads
ENV FB_USERNAME=admin
ENV FB_PASSWORD=admin

RUN apk add --no-cache curl ca-certificates

ARG TARGETARCH
RUN case "${TARGETARCH}" in \
        amd64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64" ;; \
        arm64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64" ;; \
        *)      echo "Unsupported arch: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    curl -L --retry 5 --fail -o /usr/local/bin/cloudflared "$CF_URL" && \
    chmod +x /usr/local/bin/cloudflared

RUN case "${TARGETARCH}" in \
        amd64)  FB_URL="https://github.com/filebrowser/filebrowser/releases/latest/download/linux-amd64-filebrowser.tar.gz" ;; \
        arm64)  FB_URL="https://github.com/filebrowser/filebrowser/releases/latest/download/linux-arm64-filebrowser.tar.gz" ;; \
        *)      echo "Unsupported arch: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    curl -L --retry 5 --fail "$FB_URL" | tar xz -C /usr/local/bin/ filebrowser && \
    chmod +x /usr/local/bin/filebrowser

RUN mkdir -p ${FILEBROWSER_ROOT} /config/filebrowser

# Custom startup script: reset DB each time to ensure admin/admin, then start services
RUN cat > /custom-init.sh << 'EOF'
#!/bin/sh
DB_PATH="/config/filebrowser/filebrowser.db"

# Force re-initialize database to avoid credential mismatch
rm -f "$DB_PATH"
filebrowser config init --database="$DB_PATH"
filebrowser config set --address=0.0.0.0 --port=$FILEBROWSER_PORT --root=$FILEBROWSER_ROOT --database="$DB_PATH"
filebrowser users add "$FB_USERNAME" "$FB_PASSWORD" --database="$DB_PATH" --perm.admin

echo "FileBrowser credentials: $FB_USERNAME / $FB_PASSWORD"
echo "FileBrowser serving: $FILEBROWSER_ROOT"

# Start FileBrowser
/usr/local/bin/filebrowser --database="$DB_PATH" &

sleep 2

echo "Starting Cloudflare tunnel for qBittorrent (port $WEBUI_PORT)..."
/usr/local/bin/cloudflared tunnel --url http://localhost:$WEBUI_PORT --no-autoupdate &

echo "Starting Cloudflare tunnel for FileBrowser (port $FILEBROWSER_PORT)..."
/usr/local/bin/cloudflared tunnel --url http://localhost:$FILEBROWSER_PORT --no-autoupdate &

exec /init
EOF
RUN chmod +x /custom-init.sh

EXPOSE 8000 6881 6881/udp 8080
ENTRYPOINT ["/custom-init.sh"]
