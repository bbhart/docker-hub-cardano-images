FROM ubuntu:18.04 AS builder

ENV CABAL_VERSION="3.2.0.0"
ENV GHC_VERSION="8.6.5"

ENV CARDANO_NODE_VERSION="1.16.0"

#
# Install build prereqs
#
RUN apt-get update && \
    apt-get install -y wget xz-utils build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux git jq wget libncursesw5 libtool autoconf

#
# Cabal
#
WORKDIR /tmp
RUN wget https://downloads.haskell.org/~cabal/cabal-install-${CABAL_VERSION}/cabal-install-${CABAL_VERSION}-x86_64-unknown-linux.tar.xz
RUN tar -xf cabal-install-${CABAL_VERSION}-x86_64-unknown-linux.tar.xz && mv -f cabal /usr/bin && cabal update



#
# GHC
#
WORKDIR /tmp
RUN wget https://downloads.haskell.org/~ghc/${GHC_VERSION}/ghc-${GHC_VERSION}-x86_64-deb9-linux.tar.xz && \
    tar -xf ghc-${GHC_VERSION}-x86_64-deb9-linux.tar.xz
RUN cd ghc-${GHC_VERSION} && ./configure && make install
# ghc binary ends up in /usr/local/bin

#
# Libsodium
#

WORKDIR /tmp
RUN git clone https://github.com/input-output-hk/libsodium && \
    cd libsodium && \
    git checkout 66f017f1 && \
    ./autogen.sh && \
    ./configure --disable-shared && \
    make && \
    make install && \
    ldconfig 


#
# Cardano
# 
WORKDIR /tmp
RUN git clone --recurse-submodules https://github.com/input-output-hk/cardano-node && \
    cd cardano-node && \
    git fetch --tags && \
    git checkout ${CARDANO_NODE_VERSION}

WORKDIR /tmp/cardano-node
RUN cabal install cardano-node cardano-cli \
    --installdir="/tmp/cardano-bin" \
    --install-method=copy \
    --overwrite-policy=always

CMD /bin/bash


# FROM alpine/latest

# LABEL maintainer="Mark Stopka <mark.stopka@perlur.cloud>"

# ENV SERVICE_NAME "cardano-node"

# RUN yum update -y && \
#     yum install -y libsodium && \
#     yum clean all && \
#     dnf clean all && \
#     rm -rf /var/cache/yum && \
#     rm -rf /var/cache/dnf

# EXPOSE 9000

# COPY --from=builder /usr/local/bin/cardano-node /usr/local/bin/
# COPY --from=builder /usr/local/bin/cardano-cli /usr/local/bin/
# COPY usr/local/lib/* /usr/local/lib/
# COPY usr/local/bin/* /usr/local/bin/

# RUN mkdir -p /srv/cardano/

# RUN useradd -c "Cardano node user" \
#             -d /srv/cardano/cardano-node/ \
#             -m \
#             -r \
#             -s /bin/nologin \
#             cardano-node

# RUN mkdir -p /var/run/cardano/
# RUN chown cardano-node.cardano-node /var/run/cardano/
# USER cardano-node

# RUN mkdir /srv/cardano/cardano-node/{etc,log,storage}/
# RUN mkdir /srv/cardano/cardano-node/etc/secrets/

# WORKDIR /srv/cardano/cardano-node/

# COPY srv/cardano/cardano-node/ /srv/cardano/cardano-node/

# HEALTHCHECK --interval=1s --timeout=2s CMD cnode-ctl healthcheck || exit 1

# ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
