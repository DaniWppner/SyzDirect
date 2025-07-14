# This Dockerfile is meant to provide an environment to compile the infrastructure from SyzDirecct
# This includes LLVM packages, which can be run from the container, and the Syzkaller fork.

FROM debian:bookworm
# Based on sample Dockerfile by LLVM Developers and syz-env Dockerfile from Syzkaller project.
LABEL maintainer="danielituswapp@gmail.com"

# Install build dependencies of LLVM.
# Update the apt's source list and include the sources of the packages.
RUN grep deb /etc/apt/sources.list | \
    sed 's/^deb/deb-src /g' >> /etc/apt/sources.list

# Install compiler, python, subversion, c++ libraries and wget,curl,etc. utilities.
RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates gnupg \
           # These are needed to build LLVM
           build-essential cmake make zlib1g wget subversion unzip git \
           # These are needed to build Linux kernel
           flex bison bc libelf-dev libssl-dev \
           #These are needed to build Syzdirect or Syzkaller
           curl libboost-dev python3 python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Install python dependencies of SyzDirect
COPY ./requirements.txt /src/requirements.txt
WORKDIR /src
RUN pip install -r requirements.txt --break-system-packages

# Install go toolchain to build Syzkaller fork
RUN curl https://dl.google.com/go/go1.24.4.linux-amd64.tar.gz | tar -C /usr/local -xz
ENV PATH /usr/local/go/bin:/gopath/bin:$PATH
ENV GOPATH /gopath
ENV GOMODCACHE /src/.cache/gomod
