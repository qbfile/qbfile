FROM debian:stretch

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
    && rm -rf /var/lib/apt/lists/*

# Set up environment variables
ENV USER=root
ENV HOME=/root
ENV DISPLAY=:1
ENV VNC_PORT=5901
ENV NOVNC_PORT=8000

# Create a startup script
RUN echo "#!/bin/bash\n\
mkdir -p ~/.vnc\n\
echo 'password123' | vncpasswd -f > ~/.vnc/passwd\n\
chmod 600 ~/.vnc/passwd\n\
vncserver :1 -geometry 1280x720 -depth 24\n\
/usr/share/novnc/utils/launch.sh --vnc localhost:5901 --listen 7860" > /start.sh

RUN chmod +x /start.sh

# Hugging Face Spaces run on port 7860
EXPOSE 8000

CMD ["/start.sh"]