FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV WEBUI_PORT=8000

# Install qbittorrent-nox and dependencies
RUN apt-get update && apt-get install -y \
    software-properties-common \
    && add-apt-repository ppa:qbittorrent-team/qbittorrent-stable -y \
    && apt-get update \
    && apt-get install -y qbittorrent-nox \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create a non-root user (optional but recommended)
RUN useradd -m -s /bin/bash qbt && \
    mkdir -p /config /downloads && \
    chown -R qbt:qbt /config /downloads

# Expose web UI port and BT ports
EXPOSE 8000 6881 6881/udp

# Start qbittorrent-nox with web UI on port 8000
CMD ["qbittorrent-nox", "--webui-port=8000", "--profile=/config"]
