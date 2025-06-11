
FROM fedora:39 AS lammps-builder

ARG NB_USER="sagemaker-user"
ARG NB_UID=1000
ARG NB_GID=100

USER root
# Install Python3, pip, and other dependencies
RUN dnf install -y \
    python3 \
    python3-pip \
    python3-devel \
    gcc \
    shadow-utils \
    lammps\
    lammps-openmpi \
    environment-modules \
    && useradd --create-home --shell /bin/bash --gid "${NB_GID}" --uid ${NB_UID} ${NB_USER} \
    && dnf clean all

ENV PATH="/usr/lib64/openmpi/bin:$PATH"

# Install JupyterLab and extensions
RUN python3 -m pip install --no-cache-dir \
    'jupyterlab>=4.0.0,<5.0.0' \
    urllib3 \
    jupyter-activity-monitor-extension \
    --ignore-installed

RUN python3 --version && \
    jupyter lab --version 

USER ${NB_UID}
ENTRYPOINT ["jupyter-lab"]
CMD ["--ServerApp.ip=0.0.0.0", "--ServerApp.port=8888", "--ServerApp.allow_origin=*", "--ServerApp.token=''", "--ServerApp.base_url=/jupyterlab/default"]