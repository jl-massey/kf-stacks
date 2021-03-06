ARG BASE_CONTAINER=jimmassey/kf-notebook-base
arg BASE_TAG=latest

FROM $BASE_CONTAINER:$BASE_TAG

ARG NB_USER="jovyan"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND noninteractive
ENV CONDA_DIR=/opt/conda \
    NB_USER=$NB_USER \
    CONDA_PREFIX=/opt/conda \
    JAVA_HOME=/opt/conda

WORKDIR /tmp

USER $NB_USER
# customize environment
COPY environment.yml /tmp/environment.yml
RUN ${CONDA_DIR}/bin/conda env update -q -f /tmp/environment.yml -n base \
&& jupyter lab build --dev-build=False --minimize=True \
&& conda clean --all -f -y \
&& rm -rf $CONDA_DIR/share/jupyter/lab/staging \
   $CONDA_DIR/share/jupyter/lab/static/*.map \
   /tmp/npm-* \
&& rm -rf $HOME/.cache/yarn \
&& fix-permissions $CONDA_DIR \
&& fix-permissions $HOME

# Set variables for CUDA to work properly on GPU host
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
ENV NVIDIA_REQUIRE_CUDA "cuda>=10.2"

# Make sure running as jovyan ih home directory
USER $NB_USER
WORKDIR $HOME
