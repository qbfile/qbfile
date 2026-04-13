FROM lscr.io/linuxserver/qbittorrent:latest

ENV WEBUI_PORT=8000
ENV TORRENTING_PORT=6881

# Install curl (Alpine)
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

# Create startup script (using heredoc, more robust)
RUN cat > /custom-init.sh << 'EOF'
#!/bin/sh
echo "Starting Cloudflare Tunnel to http://localhost:8000"
/usr/local/bin/cloudflared tunnel --url http://localhost:8000 --no-autoupdate &
sleep 2
exec /init
EOF
RUN chmod +x /custom-init.sh

EXPOSE 8000 6881 6881/udp
ENTRYPOINT ["/custom-init.sh"]
