# Dockerfile for omero server
FROM rockylinux:9

# Following https://docs.openmicroscopy.org/omero/5.6.3/sysadmins/unix/server-debian10-ice36.html

# Enable EPEL for dumb-init and update
RUN dnf install -y epel-release && dnf update -y
RUN dnf install -y \
    sudo \
    unzip \
    wget \
    bc \
    java-11-openjdk-headless \
    ca-certificates \
    dumb-init \
    hdf5-devel \
    python3.11 \
    python3.11-devel \
    postgresql \
    vim

# install zeroc-ice runtime dependencies
RUN sudo dnf config-manager --set-enabled crb
RUN dnf install -y \
    gcc \
    gcc-c++ \
    make \
    libdb-utils \
    bzip2-devel \
    libdb-cxx-devel \
    libdb-devel \
    expat-devel \
    openssl-devel \
    zlib-devel \
    pkgconf-pkg-config

# download and place ice files
RUN cd /tmp
RUN wget -q https://github.com/glencoesoftware/zeroc-ice-rhel9-x86_64/releases/download/20231130/Ice-3.6.5-rhel9-x86_64.tar.gz
RUN tar xf Ice-3.6.5-rhel9-x86_64.tar.gz
RUN mv Ice-3.6.5 /opt

ADD omero-ice36.env .
RUN cat omero-ice36.env >> /etc/profile

# install python dependencies
RUN python3.11 -mvenv /opt/omero/server/server_venv
RUN /opt/omero/server/server_venv/bin/pip install --upgrade pip
RUN /opt/omero/server/server_venv/bin/pip install wheel setuptools
RUN /opt/omero/server/server_venv/bin/pip install https://github.com/glencoesoftware/zeroc-ice-py-linux-x86_64/releases/download/20231130/zeroc_ice-3.6.5-cp311-cp311-manylinux_2_28_x86_64.whl
RUN /opt/omero/server/server_venv/bin/pip install omero-py==5.22.0
RUN /opt/omero/server/server_venv/bin/pip install omego
RUN /opt/omero/server/server_venv/bin/pip install omero-certificates==0.4.0
RUN /opt/omero/server/server_venv/bin/pip install omero-server
RUN /opt/omero/server/server_venv/bin/pip install omero-cli-transfer==1.3.1
RUN /opt/omero/server/server_venv/bin/pip install xsdata==26.2
# needed for OMERO.figure exports:
RUN /opt/omero/server/server_venv/bin/pip install reportlab markdown pillow

ARG UID
# `docker build --build-arg uid=63002` for prod, 63003 for dev
RUN groupadd -g 10000 jaxuserpod
RUN useradd -ms /bin/bash -g 10000 -u $UID omero-server

WORKDIR /opt/omero/server
ENV SERVER=https://downloads.openmicroscopy.org/omero/5.6.17/server-ice36.zip
RUN wget -q $SERVER -O OMERO.server-ice36.zip
RUN unzip -q OMERO.server*
RUN chown -R omero-server OMERO.server-*
RUN ln -s OMERO.server-*/ OMERO.server
RUN chown -R omero-server /opt

ENV STEWARD=https://github.com/glencoesoftware/omero-pc-steward/releases/download/v0.1.0/omero-pc-steward-0.1.0.jar
RUN wget -q $STEWARD -O /opt/omero/server/OMERO.server/lib/server/omero-pc-steward.jar
ENV LOGSED='/<logger name="ome.adapters" level="ERROR"\/>/a \ \ <!-- omero-pc-steward memory cleanup logger -->\n  <logger name="com.glencoesoftware" level="INFO"\/>'
RUN sed -i "$LOGSED" /opt/omero/server/OMERO.server/etc/logback.xml

# Startup scripts, from omero-server-docker
ADD entrypoint.sh /usr/local/bin/
COPY startup /startup/

USER omero-server
ENV OMERODIR /opt/omero/server/OMERO.server

# Copied from omero-ice36.env because /etc/profile not useful for kub pod access, just setting these env for omero-server
ENV ICE_HOME /opt/Ice-3.6.5
ENV PATH "$ICE_HOME/bin:$PATH"		
ENV SLICEPATH "$ICE_HOME/slice"

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
