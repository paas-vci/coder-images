FROM ubuntu:22.04

SHELL ["/bin/bash", "-c"]
ENV DEBIAN_FRONTEND noninteractive
ARG HOME=/home/vf-user

# Install the Docker apt repository
RUN apt-get update && \
    apt-get upgrade --yes && \
    apt-get install --yes ca-certificates

# Install baseline packages
RUN apt-get update && \
    apt-get install --yes \
    bash \
    build-essential \
    ca-certificates \
    curl \
    htop \
    locales \
    man \
    python3 \
    python3-pip \
    software-properties-common \
    sudo \
    systemd \
    systemd-sysv \
    unzip \
    vim \
    wget \
    rsync && \
    # Install latest Git using their official PPA
    add-apt-repository ppa:git-core/ppa && \
    apt-get install --yes git \

# Make typing unicode characters in the terminal work.
ENV LANG en_US.UTF-8

# Add a user `vf-user` so that you're not developing as the `root` user
RUN useradd vf-user \
    --create-home \
    --shell=/bin/bash \
    --uid=1001 \
    --user-group && \
    echo "vf-user ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers.d/nopasswd

RUN pip install numpy
RUN chown -R vf-user $HOME
RUN echo -e -n "newpasswd\nnewpasswd" | passwd vf-user
ARG DEBCONF_NONINTERACTIVE_SEEN=true
ARG DEBIAN_FRONTEND="noninteractive"
ARG TURBOVNC_VERSION=2.2.5
ARG VIRTUALGL_VERSION=2.6.4
ARG LIBJPEG_VERSION=2.0.5

RUN echo 'tzdata tzdata/Areas select Etc' | debconf-set-selections; \
    echo 'tzdata tzdata/Zones/Etc select UTC' | debconf-set-selections

RUN apt-get update && apt-get install -y \
    wget \
    unzip \
    zsh \
    supervisor \
    xorg \
    ssh \
    xfce4 \
    xfce4-goodies \
    x11-apps \
    dbus-x11 \
    xterm \
    firefox \
    fonts-lyx \
    libxtst6 \
    libxv1 \
    libglu1-mesa \
    libc6-dev \
    libglu1 \
    libsm6 \
    libxv1 \
    x11-xkb-utils \
    xauth \
    xfonts-base \
    xkb-data

# Install quality of life packages.
RUN yes | unminimize

# Remove packages which may not behave well in a VNC environment.
RUN apt-get remove -y \
    xfce4-battery-plugin \
    xfce4-power-manager-plugins \
    xfce4-pulseaudio-plugin \
    light-locker

RUN locale-gen en_US.UTF-8


ARG VNC_ROOT_DIR=/opt/vnc

RUN cd /tmp \
    && curl -fsSL -O https://netix.dl.sourceforge.net/project/turbovnc/${TURBOVNC_VERSION}/turbovnc_${TURBOVNC_VERSION}_amd64.deb \
    -O https://netix.dl.sourceforge.net/project/libjpeg-turbo/${LIBJPEG_VERSION}/libjpeg-turbo-official_${LIBJPEG_VERSION}_amd64.deb \
    -O https://netix.dl.sourceforge.net/project/virtualgl/${VIRTUALGL_VERSION}/virtualgl_${VIRTUALGL_VERSION}_amd64.deb \
    && dpkg -i *.deb \
    && rm -f /tmp/*.deb \
    && sed -i 's/$host:/unix:/g' /opt/TurboVNC/bin/vncserver

RUN ln -s /opt/TurboVNC/bin/* /opt/VirtualGL/bin/* /usr/local/bin/
# Configure VGL for use in a single user environment.
# This may trigger a warning about display managers needing to be restarted.
# This can be ignored as the VNC server manages this lifecycle.  
RUN vglserver_config -config +s +f +t

COPY turbovncserver-security.conf /etc/turbovncserver-security.conf

ENV VNC_SCRIPTS=$VNC_ROOT_DIR/scripts \
    VNC_SETUP_SCRIPTS=$VNC_ROOT_DIR/setup \
    VNC_LOG_DIR=$HOME/.vnc/log \
    VNC_XSTARTUP=$VNC_ROOT_DIR/xstartup \
    VNC_SUPERVISOR_CONFIG=$VNC_ROOT_DIR/supervisord.conf \
    VNC_PORT=5990 \
    VNC_DISPLAY_ID=:90 \
    VNC_COL_DEPTH=24 \
    VNC_RESOLUTION=3840x2160 \
    NO_VNC_HOME=$VNC_ROOT_DIR/noVNC \
    NO_VNC_PORT=6081 \
    XFCE_BASE_DIR=$VNC_ROOT_DIR/xfce4 \
    XFCE_DEST_DIR=$HOME/.config/xfce4

WORKDIR $HOME

# Enable better defaults for command tab completion.
RUN chsh -s $(readlink -f $(which zsh)) vf-user 

ADD --chown=vf-user:vf-user ./xfce4 $XFCE_BASE_DIR
ADD --chown=vf-user:vf-user ./vnc $VNC_ROOT_DIR
ADD --chown=vf-user:vf-user ./supervisor /etc/supervisor

RUN find $VNC_SETUP_SCRIPTS -name '*.sh' -exec chmod a+x {} +

RUN $VNC_SETUP_SCRIPTS/set_user_permission.sh $VNC_ROOT_DIR \
    && chmod +x $VNC_XSTARTUP

# Add vf-user-specific scripts and metadata to the image
COPY ["./vf-user", "/vf-user"]
RUN chmod +x /vf-user/configure

USER vf-user
RUN pip install numpy
RUN ls ${VNC_SETUP_SCRIPTS}
RUN $VNC_SETUP_SCRIPTS/no_vnc.sh

EXPOSE $NO_VNC_PORT
