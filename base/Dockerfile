# BSD 2-Clause License

# Copyright (c) 2020, Supreeth Herle
# All rights reserved.

# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:

# 1. Redistributions of source code must retain the above copyright notice, this
#    list of conditions and the following disclaimer.

# 2. Redistributions in binary form must reproduce the above copyright notice,
#    this list of conditions and the following disclaimer in the documentation
#    and/or other materials provided with the distribution.

# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

FROM ubuntu:focal as builder

ENV DEBIAN_FRONTEND=noninteractive

# Install updates and dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        python3-pip \
        python3-setuptools \
        python3-wheel \
        ninja-build \
        build-essential \
        flex \
        bison \
        git \
        cmake \
        libsctp-dev \
        libgnutls28-dev \
        libgcrypt-dev \
        libssl-dev \
        libidn11-dev \
        libmongoc-dev \
        libbson-dev \
        libyaml-dev \
        meson \
        mongodb \
        curl \
        gnupg \
        ca-certificates \
        libmicrohttpd-dev \
        libcurl4-gnutls-dev \
        libnghttp2-dev \
        libtins-dev \
        libidn11-dev \
        libtalloc-dev

RUN mkdir -p /etc/apt/keyrings
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg && \
    NODE_MAJOR=20 && \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && apt install nodejs -y

# Get open5gs code and install
RUN git clone --recursive https://gitlab.tech.orange/win/oinis/olb-2g6/open5gs.git && cd open5gs && \
    && meson build --prefix=`pwd`/install && \
    ninja -C build && cd build && ninja install && \
    mkdir -p /open5gs/install/include

# Building WebUI of open5gs
RUN cd open5gs/webui && npm ci

# Build final image
FROM ubuntu:focal

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        ca-certificates \
        libssl-dev \
        libyaml-dev \
        libmicrohttpd-dev \
        libmongoc-dev \
        libsctp-dev \
        libcurl4-gnutls-dev \
        libtins-dev \
        libidn11-dev \
        libtalloc-dev \
        netbase \
        ifupdown \
        net-tools \
        iputils-ping \
        python3-setuptools \
        python3-wheel \
        python3-pip \
        iptables \
        iperf \
        mongodb-clients \
        gnupg && \
    apt-get autoremove -y && apt-get autoclean

RUN pip3 install click

RUN addgroup -g 1000 open5gs && adduser -D -G open5gs open5gs -u 1000 -s /bin/sh \
    && mkdir /open5gs && chwon open5gs:open5gs /open5gs

USER 1000 

RUN mkdir -p /etc/apt/keyrings
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg && \
    NODE_MAJOR=20 && \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && apt install nodejs -y

RUN update-ca-certificates

WORKDIR /open5gs



COPY --chown=open5gs:open5gs --from=builder /open5gs/install/bin /open5gs/install/bin
COPY --chown=open5gs:open5gs --from=builder /open5gs/install/etc /open5gs/install/etc
COPY --chown=open5gs:open5gs --from=builder /open5gs/install/include /open5gs/install/include
COPY --chown=open5gs:open5gs --from=builder /open5gs/install/lib /open5gs/install/lib
COPY --chown=open5gs:open5gs --from=builder/open5gs/webui /open5gs/webui
COPY --chown=open5gs:open5gs --from=builder /open5gs/misc/db /open5gs/misc/db

COPY --chown=open5gs:open5gs grafana /open5gs/CNF/grafana
COPY --chown=open5gs:open5gs hss /open5gs/CNF/hss
COPY --chown=open5gs:open5gs metrics /open5gs/CNF/metrics
COPY --chown=open5gs:open5gs mme /open5gs/CNF/mme
COPY --chown=open5gs:open5gs sgwu /open5gs/CNF/sgwu
COPY --chown=open5gs:open5gs pcrf /open5gs/CNF/pcrf
COPY --chown=open5gs:open5gs sgwc /open5gs/CNF/sgwc

# Set the working directory to open5gs


COPY open5gs_init.sh /
CMD /open5gs_init.sh
