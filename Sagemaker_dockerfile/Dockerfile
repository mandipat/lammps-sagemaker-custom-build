FROM public.ecr.aws/sagemaker/sagemaker-distribution:latest-gpu

ARG NB_USER="sagemaker-user"
ARG NB_UID=1000
ARG NB_GID=100

USER root
RUN apt-get update && apt-get install -y --no-install-recommends \

COPY ../lammps_bundle_gpu/lmp /usr/local/bin/lmp_gpu
RUN chmod +x /usr/local/bin/lmp_gpu

COPY ../lammps_bundle_cpu/bin/lmp_cpu /usr/local/bin/lmp
RUN chmod +x /usr/local/bin/lmp

USER $NB_USER

ENTRYPOINT ["jupyter-lab"]
CMD ["--ServerApp.ip=0.0.0.0", "--ServerApp.port=8888", "--ServerApp.allow_origin=*", "--ServerApp.token=''", "--ServerApp.base_url=/jupyterlab/default"]