# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG BASE_CONTAINER=alaurans/scipy-notebook
FROM $BASE_CONTAINER

USER root

# R pre-requisites
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    fonts-dejavu \
    gfortran \
    gcc \
    r-base \
    r-base-core \
    r-recommended \
    r-base-dev xml2 libxml2-dev libssl-dev libcurl4-openssl-dev unixodbc-dev \
    && \
    rm -rf /var/lib/apt/lists/*


# Julia dependencies
# install Julia packages in /opt/julia instead of $HOME
ENV JULIA_DEPOT_PATH=/opt/julia \
    JULIA_PKGDIR=/opt/julia \
    JULIA_VERSION=1.3.1

RUN mkdir /opt/julia-${JULIA_VERSION} && \
    cd /tmp && \
    wget -q https://julialang-s3.julialang.org/bin/linux/x64/`echo ${JULIA_VERSION} | cut -d. -f 1,2`/julia-${JULIA_VERSION}-linux-x86_64.tar.gz && \
    echo "faa707c8343780a6fe5eaf13490355e8190acf8e2c189b9e7ecbddb0fa2643ad *julia-${JULIA_VERSION}-linux-x86_64.tar.gz" | sha256sum -c - && \
    tar xzf julia-${JULIA_VERSION}-linux-x86_64.tar.gz -C /opt/julia-${JULIA_VERSION} --strip-components=1 && \
    rm /tmp/julia-${JULIA_VERSION}-linux-x86_64.tar.gz
RUN ln -fs /opt/julia-*/bin/julia /usr/local/bin/julia

# Create JULIA_PKGDIR \
RUN mkdir $JULIA_PKGDIR && \
    chown $NB_USER $JULIA_PKGDIR && \
    fix-permissions $JULIA_PKGDIR

USER $NB_UID

RUN julia -e 'import Pkg; Pkg.update()' && \
    julia -e 'import Pkg; Pkg.add("HDF5")' && \
    julia -e "using Pkg; pkg\"add IJulia\"; pkg\"precompile\""

# R packages including IRKernel which gets installed globally.
ENV R_LIBS_USER=${HOME}/.local/R/library

RUN mkdir -p ${R_LIBS_USER} &&  \
    R -e 'install.packages(c("caret", "crayon", "devtools", "forecast", "hexbin", "htmltools", "htmlwidgets", "nycflights13", "plyr", "randomforest", "rcurl", "reshape2", "rmarkdown", "rsqlite", "shiny", "tidyverse", "IRkernel"), dependencies=TRUE)' && \
    R -e 'IRkernel::installspec()'
