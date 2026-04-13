FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Install desktop environment and NoVNC
RUN apt-get update && apt-get install -y \
    xfce4 \
    xfce4-goodies \
    tightvncserver \
    novnc \
    websockify \
    curl \
    procps \
    python3 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV USER=root
ENV HOME=/root
ENV DISPLAY=:1

# Create a startup script with the CORRECT novnc path
RUN echo "#!/bin/bash\n\
mkdir -p ~/.vnc\n\
# VNC passwords are limited to 8 characters\n\
echo 'pass1234' | vncpasswd -f > ~/.vnc/passwd\n\
chmod 600 ~/.vnc/passwd\n\
# Stop any existing sessions\n\
vncserver -kill :1 || true\n\
# Start VNC\n\
vncserver :1 -geometry 1280x720 -depth 24\n\
# Start NoVNC proxy (Correct path for Ubuntu 22.04 is /usr/bin/novnc_proxy)\n\
/usr/bin/novnc_proxy --vnc localhost:5901 --listen 8000" > /start.sh

RUN chmod +x /start.sh

EXPOSE 8000

CMD ["/start.sh"]