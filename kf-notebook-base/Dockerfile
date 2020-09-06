# kf-notebook-base
ARG BASE_CONTAINER=ubuntu:20.04

FROM $BASE_CONTAINER

ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
USER root

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update --fix-missing && apt-get upgrade -yq\
&& apt-get install -yq --no-install-recommends \
   apt-transport-https \
   bzip2 \
   ca-certificates \
   curl \
   fonts-liberation \
   git \
   libaio1 \
   locales \
   lsb-release \
   nano \
   run-one \
   sudo \
   unzip \
   wget \
   zip \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
ENV PATH=$CONDA_DIR/bin:$PATH \
    HOME=/home/$NB_USER

COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions \
&& sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc

RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su \
 && sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers \
 && sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers \
 && useradd -m -s /bin/bash -N -u $NB_UID $NB_USER \
 && mkdir -p $CONDA_DIR \
 && chown $NB_USER:$NB_GID $CONDA_DIR \
 && chmod g+w /etc/passwd \
 && fix-permissions $HOME \
 && fix-permissions $CONDA_DIR

USER $NB_UID
WORKDIR $HOME
ARG PYTHON_VERSION=default

RUN mkdir $HOME/work \
&& fix-permissions $HOME

ENV MINICONDA_VERSION=4.8.3 \
    MINICONDA_MD5=751786b92c00b1aeae3f017b781018df \
    CONDA_VERSION=4.8.4

WORKDIR /tmp

RUN wget --quiet https://repo.continuum.io/miniconda/Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh \
&& echo "${MINICONDA_MD5} *Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh" | md5sum -c - \
&& /bin/bash Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p $CONDA_DIR \
&& rm Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh \
# && echo "conda ${CONDA_VERSION}" >> $CONDA_DIR/conda-meta/pinned \
&& conda config --system --prepend channels conda-forge \
&& conda config --system --set auto_update_conda false \
&& conda config --system --set show_channel_urls true \
# && conda config --system --set channel_priority strict \
&& if [ ! $PYTHON_VERSION = 'default' ]; then conda install --yes python=$PYTHON_VERSION; fi \
# && conda list python | grep '^python ' | tr -s ' ' | cut -d '.' -f 1,2 | sed 's/$/.*/' >> $CONDA_DIR/conda-meta/pinned \
&& conda install --quiet --yes conda \
&& conda install --quiet --yes pip \
&& conda update --all --quiet --yes \
&& conda clean --all -f -y \
&& rm -rf $HOME/.cache/yarn \
&& fix-permissions $CONDA_DIR \
&& fix-permissions $HOME


USER $NB_USER
RUN echo ". ${CONDA_DIR}/etc/profile.d/conda.sh" >> ${HOME}/.bashrc \
&& echo "conda activate base" >> ${HOME}/.bashrc

# Install Tini
RUN conda install --quiet --yes 'tini=0.18.0' \
&& conda list tini | grep tini | tr -s ' ' | cut -d ' ' -f 1,2 >> $CONDA_DIR/conda-meta/pinned \
&& conda clean --all -f -y \
&& fix-permissions $CONDA_DIR \
&& fix-permissions $HOME

# Install jupyter notebook, lab and hub
RUN conda install --quiet --yes \
   notebook=6.1 \
   jupyterlab=2.2 \
   jupyterhub \
&& conda clean --all -f -y \
&& npm cache clean --force \
&& jupyter notebook --generate-config \
&& rm -rf $CONDA_DIR/share/jupyter/lab/staging \
&& rm -rf $HOME/.cache/yarn \
&& fix-permissions $CONDA_DIR \
&& fix-permissions $HOME

# customize jupyterlab
RUN ${CONDA_DIR}/bin/conda install -c conda-forge -c defaults \
   elyra \
   nb_conda_kernels \
   jupyterlab-nvdashboard \
&& jupyter lab build --dev-build=False --minimize=True \
&& conda clean --all -f -y \
&& rm -rf $CONDA_DIR/share/jupyter/lab/staging \
   $CONDA_DIR/share/jupyter/lab/static/*.map \
   /tmp/npm-* \
&& rm -rf $HOME/.cache/yarn \
&& fix-permissions $CONDA_DIR \
&& fix-permissions $HOME

USER root
RUN ln -s $CONDA_DIR/bin/conda /etc/profile.d/conda.sh \
&&  ln -s $CONDA_DIR /opt/anaconda
RUN echo "$NB_USER ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers

ENV NB_PREFIX /

EXPOSE 8888
EXPOSE 8787
EXPOSE 8786

ENTRYPOINT ["tini", "-g", "--"]
CMD ["sh","-c", "jupyter lab --notebook-dir=${HOME} --ip=0.0.0.0 --no-browser --allow-root --port=8888 --NotebookApp.token='' --NotebookApp.password='' --NotebookApp.allow_origin='*' --NotebookApp.base_url=${NB_PREFIX}"]

# Make sure running as jovyan ih home directory
USER $NB_UID
WORKDIR $HOME
