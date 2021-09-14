FROM ubuntu:20.04

ARG build_version

RUN echo "=====> Building Zeek Version $build_version..."

USER root

RUN echo "=====> Updating packages..."
RUN export DEBIAN_FRONTEND=noninteractive && apt-get update && apt-get install --no-install-recommends -y \
    bash \
    cmake \
    make \
    curl \
    gcc \
    g++ \
    flex \
    bison \
    libpcap-dev \
    python3 \
    python3-dev \
    python3-pip \
    python3-setuptools \
    git \
    libssl-dev \
    swig \
    jq \
    zlib1g-dev \
    bsdmainutils \ 
    software-properties-common \
    less \
    zsh \
    tzdata \
    vim \
    linux-headers-$(uname -r) \
    && rm -rf /var/lib/apt/lists/*

# set your timezone
RUN ln -fs /usr/share/zoneinfo/America/Denver /etc/localtime
RUN dpkg-reconfigure --frontend noninteractive tzdata

# Install oh-my-zsh
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# Install additional VIM plugins
RUN git clone --depth=1 https://github.com/amix/vimrc.git ~/.vim_runtime && \
    sh ~/.vim_runtime/install_awesome_vimrc.sh

# Install the VIM Zeek plugin
RUN cd ~/.vim_runtime && \
    git clone https://github.com/mephux/bro.vim.git my_plugins/bro-vim && \
    echo "syntax on\ncolor torte\nset nu" > my_configs.vim

RUN pip3 install setuptools --upgrade && pip3 install --upgrade pip

RUN mkdir -p /app/zeek-source && mkdir /opt/zeek

WORKDIR /app/zeek-source

# Download and compile from Zeek source
# This will need to be updated when the Zeek version changes
# Doing this so we match the Zeek version running on Corelight
RUN echo "=====> Compiling Zeek..."
RUN curl -LJO https://download.zeek.org/zeek-${build_version}.tar.gz && tar xzvf zeek-${build_version}.tar.gz

WORKDIR /app/zeek-source/zeek-${build_version}

RUN ./configure --prefix=/opt/zeek \
     --build-type=MinSizeRel \
     --disable-broker-tests \
     --disable-zeekctl \
     --disable-python \
     --disable-auxtools \
     && make -j 2 && make install && make clean

RUN pip3 install zkg

ENV ZEEKPATH .:/opt/zeek/share/zeek:/opt/zeek/share/zeek/policy:/opt/zeek/share/zeek/site
ENV PATH /opt/zeek/bin:$PATH

# Setup Zeek environment for local persistence
RUN echo "export ZEEKPATH=$ZEEKPATH:/app/zeek-work/zeek/share/zeek/site" >> ~/.bashrc && \
    echo "export ZEEK_SCRIPT_DIR=$ZEEKPATH:/app/zeek-work/zeek/lib/zeek/plugins" >> ~/.bashrc

# Configure bro-pkg/zkg
RUN zkg autoconfig --force

# Configure ZSH to be default
RUN chsh -s $(which zsh) && sed -i 's/ZSH_THEME=.*/ZSH_THEME="af-magic"/' ~/.zshrc

# EXPOSE 8081

WORKDIR /app/zeek-source

# Compile AF_PACKET plugin
#RUN git clone https://github.com/J-Gras/bro-af_packet-plugin.git \
#  && cd bro-af_packet-plugin \
#  && find . -name "*.bro" -exec sh -c 'mv "$1" "${1%.bro}.zeek"' _ {} \; \
#  && ./configure --with-kernel=/usr --bro-dist=/app/zeek-source/zeek-${build_version} \
#  && make \
#  && make install \
#  && zeek -NN Bro::AF_Packet

#RUN git clone https://github.com/J-Gras/zeek-af_packet-plugin 
#RUN cd zeek-af_packet-plugin \
#  && ./configure \
#  && make \
#  && make install \
#  && zeek -NN Zeek::AF_Packet

RUN echo "Compiling AF_PACKET plugin..."
RUN zkg install --force zeek-af_packet-plugin
RUN zeek -NN Zeek::AF_Packet

WORKDIR /app/zeek-work

# Use custom copy of local.bro to enable special configs such as JSON logs
# Also enables use of the Zeek package installer to build code bundles
COPY local.zeek /opt/zeek/share/zeek/site/local.zeek
#COPY node.cfg /opt/zeek/etc/node.cfg
#COPY networks.cfg /opt/zeek/etc/networks.cfg
#COPY zeekctl.cfg /opt/zeek/etc/zeekctl.cfg
# COPY cessoc.bundle /app/zeek-work

RUN echo "=====> Installing Zeek packages..."
# RUN zkg install --force zeek/corelight/json-streaming-logs
RUN zkg install --force \
    salesforce/ja3 \
    salesforce/hassh \
    mitre-attack/bzar \
    https://github.com/nturley3/json-streaming-logs

WORKDIR /opt/zeek/logs

# Set the entrypoint and output some helpful environment info
ENTRYPOINT echo "Zeek version: " && \
           zeek -v && \ 
           echo "\nzkg config: " && \
           zkg config && \
           echo "Display Export: " && \
           echo $DISPLAY && \
           /bin/zsh


