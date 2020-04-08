# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.

# Ubuntu 18.04 (bionic)
# https://hub.docker.com/_/ubuntu/?tab=tags&name=bionic
# OS/ARCH: linux/amd64
#ARG ROOT_CONTAINER=ubuntu:bionic-20200219@sha256:0925d086715714114c1988f7c947db94064fd385e171a63c07730f1fa014e6f9
ARG ROOT_CONTAINER=nvidia/cuda:10.0-cudnn7-runtime
ARG BASE_CONTAINER=$ROOT_CONTAINER
FROM $BASE_CONTAINER

ARG NB_USER="datascientist"
ARG NB_UID="1000"
ARG NB_GID="1000"

USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update \
 && apt-get install -yq --no-install-recommends \
    bzip2 \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
    run-one \
    git \
    curl \
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libffi-dev \
    libsqlite3-dev \
    liblzma-dev \
 && curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash - \
 && apt-get install -yq --no-install-recommends nodejs \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Configure environment
ENV SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    HOME=/home/$NB_USER \
    TINI_VERSION=v0.18.0

# Copy a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /usr/local/bin/tini
RUN chmod a+rx /usr/local/bin/fix-permissions /usr/local/bin/tini

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc

# Create NB_USER wtih name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    chmod g+w /etc/passwd && \
    fix-permissions $HOME

USER $NB_UID
WORKDIR $HOME
ARG PYTHON_VERSION=3.8.2

# Setup work directory for backward-compatibility
RUN mkdir /home/$NB_USER/work 

ENV PYENV_ROOT=$HOME/.pyenv
ENV PATH=$PYENV_ROOT/bin:${PYENV_ROOT}/shims:$PATH \
    PYTHONDONTWRITEBYTECODE=true

RUN cd ${HOME} && \
    curl https://pyenv.run | bash && \
    echo "export PATH=\"${PYENV_ROOT}/bin:\$PATH\"" >> .bashrc && \
    echo "eval \"\$(pyenv init -)\"" >> .bashrc && \
    echo "eval \"\$(pyenv virtualenv-init -)\"" >> .bashrc && \
    pyenv install ${PYTHON_VERSION} && pyenv global ${PYTHON_VERSION}


RUN pip install --upgrade pip setuptools && \
    curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python && \
    echo "export PATH=\"${HOME}/.poetry/bin:\$PATH\"" >> .bashrc && \
    . $HOME/.poetry/env && \
    poetry config virtualenvs.create false

# Install Jupyter Notebook, Lab, and Hub
# Generate a notebook server config
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
RUN pip install --no-cache \
    notebook \
    jupyterhub \
    jupyterlab && \
    npm cache clean --force && \
    jupyter notebook --generate-config

EXPOSE 8888

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh"]

# Copy local files as late as possible to avoid cache busting
COPY start.sh start-notebook.sh start-singleuser.sh /usr/local/bin/
COPY jupyter_notebook_config.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root
RUN fix-permissions /etc/jupyter/

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
