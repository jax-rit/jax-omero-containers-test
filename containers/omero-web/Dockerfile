# Dockerfile for an omero web client
FROM rockylinux:9

# Following https://github.com/ome/omero-web-docker

# Enable EPEL for dumb-init and update
RUN dnf install -y epel-release && dnf update -y
RUN dnf install -y unzip python3.11 python3.11-devel dumb-init zlib-devel libjpeg-turbo-devel gcc gcc-c++ make

RUN useradd -ms /bin/bash omero-web
RUN mkdir -p /opt/omero/web/OMERO.web/etc/grid && \
    chown -R omero-web /opt/omero/web/OMERO.web

RUN python3.11 -m venv /opt/omero/web/venv3
RUN /opt/omero/web/venv3/bin/pip install --upgrade pip
RUN /opt/omero/web/venv3/bin/pip install wheel

# TODO: Download & check hash function
RUN /opt/omero/web/venv3/bin/pip install --upgrade https://github.com/glencoesoftware/zeroc-ice-py-linux-x86_64/releases/download/20231130/zeroc_ice-3.6.5-cp311-cp311-manylinux_2_28_x86_64.whl
RUN /opt/omero/web/venv3/bin/pip install omero-py==5.22.0
RUN /opt/omero/web/venv3/bin/pip install omero-web==5.31.0

RUN /opt/omero/web/venv3/bin/pip install \
        omero-figure \
        omero-iviewer \
        whitenoise \
        omero-gallery \
        django-csp

# Startup
ADD entrypoint.sh /usr/local/bin/
RUN chmod 755 /usr/local/bin/entrypoint.sh
COPY startup /startup/
ADD 01-default-webapps.omero /opt/omero/web/config/

USER omero-web

ENV OMERODIR=/opt/omero/web/OMERO.web/
ENV CONFIG_omero_web_application__server_host=0.0.0.0
# ENV CONFIG_omero_web_debug=true
# ENV CONFIG_omero_web_server__list='[["host.docker.internal", 6064, "host.docker.internal"]]'

EXPOSE 4080
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
