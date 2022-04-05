FROM alpine:3.13 as builder

ARG build_version

ENV ZEEK_VERSION $build_version
RUN echo "===> Building Zeek $build_version"

RUN apk add --no-cache zlib openssl libstdc++ libpcap libgcc
RUN apk add --no-cache -t .build-deps \
  bsd-compat-headers \
  libmaxminddb-dev \
  linux-headers \
  openssl-dev \
  libpcap-dev \
  python3-dev \
  zlib-dev \
  binutils \
  fts-dev \
  cmake \
  clang \
  bison \
  bash \
  swig \
  perl \
  make \
  flex \
  git \
  g++ \
  fts \
  python3 \
  py3-pip

RUN echo "===> Cloning zeek" \
  && cd /tmp \
  && git clone --recursive --branch v$ZEEK_VERSION https://github.com/zeek/zeek.git

RUN echo "===> Compiling zeek" \
  && cd /tmp/zeek \
  && CC=clang ./configure --prefix=/usr/local/zeek \
  --build-type=MinSizeRel \
  --disable-broker-tests \
  --disable-zeekctl \
  --disable-auxtools \
  --disable-python \
  && make -j 2 \
  && make install

RUN pip3 install --upgrade pip

# Zeek 4.0.0 now includes zkg bundled
# See https://docs.zeek.org/projects/package-manager/en/stable/quickstart.html
# We still need to install the following dependencies for zkg
RUN pip3 install gitpython semantic-version

ENV ZEEKPATH .:/usr/local/zeek/share/zeek:/usr/local/zeek/share/zeek/policy:/usr/local/zeek/share/zeek/site
ENV PATH /usr/local/zeek/bin:$PATH

# Issues: https://github.com/J-Gras/zeek-af_packet-plugin/issues/24
# Having difficulties getting AF_PACKET compiled either through zkg or source compile
# Install AF_PACKET
#RUN echo "===> Installing AF_PACKET plugin" \
#  && zkg install --force zeek-af_packet-plugin --version master \
#  && /usr/local/zeek/bin/zeek -NN Zeek::AF_Packet
# Doesn't yet work due to bug. See github. 

#RUN echo "===> Compiling AF_PACKET plugin..." \
#  && mkdir -p /tmp/zeek/aux \
#  && cd /tmp/zeek/aux/ \
#  && git clone --depth 1 https://github.com/J-Gras/zeek-af_packet-plugin.git \
#  && cd /tmp/zeek/aux/zeek-af_packet-plugin \
#  && CC=clang ./configure --with-kernel=/usr --zeek-dist=/tmp/zeek \
#  && make -j 2 \
#  && make install \
#  && /usr/local/zeek/bin/zeek -NN Zeek::AF_Packet

RUN echo "===> Compiling AF_PACKET plugin..." \
   && mkdir -p /tmp/zeek/aux \
   && cd /tmp/zeek/aux \
   && git clone https://github.com/J-Gras/zeek-af_packet-plugin \
   && cd /tmp/zeek/aux/zeek-af_packet-plugin \
   && sed -i 's/build_command.*/build_command = .\/configure --with-kernel=\/usr \&\& make/' ./zkg.meta \
   && git add zkg.meta \
   && git config user.name "temp" \
   && git config user.email "temp@temp.com" \
   && git commit -m "bug fix" \
   && cd /tmp/zeek/aux \
   && zkg install --force ./zeek-af_packet-plugin \
   && /usr/local/zeek/bin/zeek -NN Zeek::AF_Packet

# Install some packages
RUN echo "===> Installing Zeek Packages..." \
  && zkg install --force \
     salesforce/ja3 \
     salesforce/hassh \
     corelight/json-streaming-logs

RUN echo "===> Shrinking image..." \
  && strip -s /usr/local/zeek/bin/zeek

RUN echo "===> Size of the Zeek install..." \
  && du -sh /usr/local/zeek
####################################################################################################
FROM alpine:3.13

RUN apk --no-cache add ca-certificates zlib openssl libstdc++ libpcap libmaxminddb libgcc fts

COPY --from=builder /usr/local/zeek /usr/local/zeek
COPY local.zeek /usr/local/zeek/share/zeek/site/local.zeek

WORKDIR /zeek

ENV ZEEKPATH .:/data/config:/usr/local/zeek/share/zeek:/usr/local/zeek/share/zeek/policy:/usr/local/zeek/share/zeek/site
ENV PATH $PATH:/usr/local/zeek/bin

ENTRYPOINT ["zeek"]
CMD ["-h"]
