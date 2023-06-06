FROM docker.io/ubuntu:22.04

#
# Use docker buildx to build this image for multiple architectures
# docker buildx build --platform linux/amd64,linux/arm64 -t  patrickglass526/sysbench:latest --push .
#

ENV DEBIAN_FRONTEND=noninteractive

# Install sysbench dependencies
RUN apt-get update && apt-get install -y \
    curl \
    gnupg2 \
    lsb-release \
    wget \
    htop \
    git make automake libtool pkg-config libaio-dev \
    libmysqlclient-dev libssl-dev \
    mysql-client \
    && rm -rf /var/lib/apt/lists/*

# Install sysbench from source
RUN git clone https://github.com/akopytov/sysbench.git && \
    cd sysbench && \
    ./autogen.sh && \
    ./configure && \
    make -j && \
    make install

# Install sysbench from package (usually older version)
# RUN apt-get update && apt-get install -y curl wget && rm -rf /var/lib/apt/lists/*
# RUN wget https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh && bash script.deb.sh
# RUN apt-get -y install sysbench

ENV MYSQL_HOST=127.0.0.1
ENV MYSQL_PS1="tidb> "
ENV MYSQL_TCP_PORT=4000