ARG BASE_CONTAINER=jimmassey/kf-notebook-base:0.1

FROM $BASE_CONTAINER

ARG NB_USER="jovyan"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
USER root

ENV DEBIAN_FRONTEND noninteractive
ENV CONDA_DIR=/opt/conda \
    NB_USER=$NB_USER

WORKDIR /tmp

USER $NB_USER
# customize environment
COPY environment.yml /tmp/environment.yml
RUN ${CONDA_DIR}/bin/conda env update -q -f /tmp/environment.yml -n base \
&& conda clean --all -f -y \
&& npm cache clean --force \
&& rm -rf $CONDA_DIR/share/jupyter/lab/staging \
&& rm -rf $HOME/$NB_USER/.cache/yarn \
&& fix-permissions $CONDA_DIR \
&& fix-permissions $HOME

USER root
# Install Oracle Instantclient
RUN wget -O instantclient-basiclite-linux.x64-19.8.0.0.0dbru.zip "https://download.oracle.com/otn_software/linux/instantclient/19800/instantclient-basiclite-linux.x64-19.8.0.0.0dbru.zip" \
&& unzip instantclient-basiclite-linux.x64-19.8.0.0.0dbru.zip -d /opt/oracle \
&& sh -c "echo /opt/oracle/instantclient_19_8 > /etc/ld.so.conf.d/oracle-instantclient.conf" \
&& ldconfig

# Make sure running as jovyan ih home directory
USER $NB_USER
WORKDIR $HOME
