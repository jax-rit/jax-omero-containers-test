# Dockerfile for omero server
FROM debian:bookworm

# Following https://docs.openmicroscopy.org/omero/5.6.3/sysadmins/unix/server-debian10-ice36.html

RUN apt-get update
RUN apt-get install -y -q \
    unzip \
    wget \
    bc \
    default-jre \
    dumb-init \
    libhdf5-serial-dev \
    python3 \
    python3-venv \
    python3-dev \
    postgresql-client \
    ca-certificates-java \
    vim

# install zeroc-ice
RUN apt-get install -y -q \
    build-essential \
    db5.3-util \
    libbz2-dev \
    libdb++-dev \
    libdb-dev \
    libexpat-dev \
    libmcpp-dev \
    libssl-dev \
    mcpp \
    zlib1g-dev \
    pkg-config

# download and place ice files
RUN cd /tmp
RUN wget -q https://github.com/ome/zeroc-ice-debian10/releases/download/0.1.0/ice-3.6.5-0.1.0-debian10-amd64.tar.gz
RUN tar xf ice-3.6.5-0.1.0-debian10-amd64.tar.gz
RUN mv ice-3.6.5-0.1.0 /opt

ADD omero-ice36.env .
RUN cat omero-ice36.env >> /etc/profile

# install python dependencies
RUN python3 -mvenv /opt/omero/server/server_venv
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
ENV ICE_HOME /opt/ice-3.6.5-0.1.0		
ENV PATH "$ICE_HOME/bin:$PATH"		
ENV SLICEPATH "$ICE_HOME/slice"

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
