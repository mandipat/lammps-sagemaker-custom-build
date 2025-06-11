FROM nvcr.io/nvidia/cuda:12.2.0-runtime-ubuntu22.04

ARG NB_USER="sagemaker-user"
ARG NB_UID=1000
ARG NB_GID=100

USER root
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone

# Install dependencies (including sudo)
RUN rm -f /etc/apt/sources.list.d/cuda*.list && \
    apt-get update && apt-get install -y --no-install-recommends --fix-missing \
    python3 \
    python3-pip \
    python3-dev \
    gcc \
    environment-modules \
    libgomp1 \
    libstdc++6 \
    libopenmpi-dev \
    openmpi-bin \
    libfftw3-dev \
    libjpeg-dev \
    libpng-dev \
    libhdf5-dev \
    libblas-dev \
    liblapack-dev \
    # Add sudo here!
    sudo \
    && useradd --create-home --shell /bin/bash --gid "${NB_GID}" --uid ${NB_UID} ${NB_USER} \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV PATH="/usr/lib64/openmpi/bin:$PATH"

COPY ../../lammps_bundle/lmp /usr/local/bin/lmp_gpu
RUN chmod +x /usr/local/bin/lmp_gpu

COPY ../../install-cpu/bin/lmp_cpu /usr/local/bin/lmp
RUN chmod +x /usr/local/bin/lmp

# Now that sudo is installed, /etc/sudoers.d/ will exist
RUN echo "sagemaker-user ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/sagemaker-nopasswd \
    && chmod 0440 /etc/sudoers.d/sagemaker-nopasswd

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