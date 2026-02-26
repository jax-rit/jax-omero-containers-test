FROM google/cloud-sdk:558.0.0

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
    vim

# install python dependencies
RUN python3 -mvenv /opt/transfer_venv
RUN /opt/transfer_venv/bin/pip install --upgrade pip
RUN /opt/transfer_venv/bin/pip install wheel setuptools
RUN /opt/transfer_venv/bin/pip install https://github.com/glencoesoftware/zeroc-ice-py-linux-x86_64/releases/download/20231130/zeroc_ice-3.6.5-cp311-cp311-manylinux_2_28_x86_64.whl
RUN /opt/transfer_venv/bin/pip install notebook
RUN /opt/transfer_venv/bin/pip install omero-cli-transfer==1.3.1
RUN /opt/transfer_venv/bin/pip install google-cloud-storage

RUN groupadd -g 10000 jaxuserpod
RUN useradd -ms /bin/bash -g 10000 -u 63003 svc-omerodev-pod
USER svc-omerodev-pod
