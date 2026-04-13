FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Install base packages + sudo + desktop + tools
RUN apt-get update && apt-get install -y \
    sudo \
    xfce4 \
    xfce4-goodies \
    tightvncserver \
    novnc \
    websockify \
    curl \
    wget \
    gnupg \
    apt-transport-https \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV USER=root
ENV HOME=/root
ENV DISPLAY=:1

# ----------------------------
# Install RustDesk
# ----------------------------
RUN curl -LO https://github.com/rustdesk/rustdesk/releases/download/1.4.6/rustdesk-1.4.6-aarch64.deb && \
    apt-get update && \
    apt-get install -y ./rustdesk-1.4.6-aarch64.deb && \
    rm rustdesk-1.4.6-aarch64.deb

# ----------------------------
# Install Firefox (Mozilla repo)
# ----------------------------
RUN install -d -m 0755 /etc/apt/keyrings && \
    wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- | tee /etc/apt/keyrings/packages.mozilla.org.asc > /dev/null && \
    echo "deb [signed-by=/etc/apt/keyrings/packages.mozilla.org.asc] https://packages.mozilla.org/apt mozilla main" > /etc/apt/sources.list.d/mozilla.list && \
    echo 'Package: *\nPin: origin packages.mozilla.org\nPin-Priority: 1000' > /etc/apt/preferences.d/mozilla && \
    apt-get update && \
    apt-get install -y firefox

# ----------------------------
# VNC setup
# ----------------------------
RUN mkdir -p ~/.vnc && \
    echo "#!/bin/bash\nstartxfce4 &" > ~/.vnc/xstartup && \
    chmod +x ~/.vnc/xstartup && \
    echo "pass1234" | vncpasswd -f > ~/.vnc/passwd && \
    chmod 600 ~/.vnc/passwd

# Startup script
RUN echo "#!/bin/bash\n\
vncserver -kill :1 || true\n\
vncserver :1 -geometry 1280x720 -depth 24\n\
websockify --web=/usr/share/novnc 8000 localhost:5901" > /start.sh

RUN chmod +x /start.sh

EXPOSE 8000

CMD ["/start.sh"]