FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Install Desktop, VNC, and Websockify (the engine for NoVNC)
RUN apt-get update && apt-get install -y \
    xfce4 \
    xfce4-goodies \
    tightvncserver \
    novnc \
    websockify \
    curl \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV USER=root
ENV HOME=/root
ENV DISPLAY=:1

# 1. Create the VNC password
# 2. Setup the Xstartup file to ensure the XFCE Desktop actually starts
# 3. Start VNC
# 4. Start Websockify directly (points port 8000 to VNC port 5901)
RUN mkdir -p ~/.vnc && \
    echo "#!/bin/bash\nstartxfce4 &" > ~/.vnc/xstartup && \
    chmod +x ~/.vnc/xstartup && \
    echo "pass1234" | vncpasswd -f > ~/.vnc/passwd && \
    chmod 600 ~/.vnc/passwd

RUN echo "#!/bin/bash\n\
vncserver -kill :1 || true\n\
vncserver :1 -geometry 1280x720 -depth 24\n\
# Directly use websockify to serve the NoVNC web files\n\
websockify --web=/usr/share/novnc 8000 localhost:5901" > /start.sh

RUN chmod +x /start.sh

# Using port 8000 because your Diploi dashboard shows it
EXPOSE 8000

CMD ["/start.sh"]