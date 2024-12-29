# Stage 1: Builder
FROM ubuntu:24.04 AS builder

# Build arguments
ARG BITCOIN_VERSION=28.0
ARG BITCOIN_URL=https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/bitcoin-${BITCOIN_VERSION}.tar.gz
ARG BITCOIN_SHA256=https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS

# Set noninteractive mode and timezone
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

# Install build dependencies
RUN apt-get update && apt-get install -y \
    automake \
    build-essential \
    libtool \
    autotools-dev \
    pkg-config \
    libssl-dev \
    libevent-dev \
    libboost-system-dev \
    libboost-filesystem-dev \
    libboost-chrono-dev \
    libboost-program-options-dev \
    libboost-test-dev \
    libboost-thread-dev \
    libminiupnpc-dev \
    libnatpmp-dev \
    libzmq3-dev \
    systemtap-sdt-dev \
    libsqlite3-dev \
    curl \
    bsdmainutils \
    && rm -rf /var/lib/apt/lists/*

# Download Bitcoin source
WORKDIR /tmp
RUN curl -O ${BITCOIN_URL} \
    && curl -O ${BITCOIN_SHA256} \
    && grep "bitcoin-${BITCOIN_VERSION}.tar.gz" SHA256SUMS | sha256sum -c \
    && tar xfz bitcoin-${BITCOIN_VERSION}.tar.gz

# Build Bitcoin Core
WORKDIR /tmp/bitcoin-${BITCOIN_VERSION}
RUN ./autogen.sh \
    && ./configure \
        --without-gui \
        --enable-reduce-exports \
        --with-natpmp \
        --with-miniupnpc \
    && make -j$(nproc) \
    && make install

# Generate bitcoin.conf sample file
RUN \
    BINDIR=/usr/local/bin \
    TOPDIR=. \
    SHARE_EXAMPLES_DIR=/usr/local/share \
    ./contrib/devtools/gen-bitcoin-conf.sh

# Stage 2: Final image
FROM ubuntu:24.04

ARG BITCOIN_VERSION

LABEL \
    org.opencontainers.image.title="Bitcoin Core Node" \
    org.opencontainers.image.description="A Bitcoin Core Node" \
    org.opencontainers.image.version="${BITCOIN_VERSION}" \
    org.opencontainers.image.url="https://github.com/salessandri/docker-bitcoind" \
    org.opencontainers.image.authors="Santiago Alessandri <san.lt.ss@gmail.com>" \
    org.opencontainers.image.licenses="MIT"

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    libevent-2.1-7t64 \
    libevent-pthreads-2.1-7t64 \
    libboost-system1.83.0 \
    libboost-filesystem1.83.0 \
    libboost-chrono1.83.0t64 \
    libboost-program-options1.83.0 \
    libboost-thread1.83.0 \
    libsqlite3-0 \
    libminiupnpc17 \
    libnatpmp1t64 \
    libzmq5 \
    && rm -rf /var/lib/apt/lists/*

# Copy binaries from builder
COPY --from=builder /usr/local/bin/bitcoind /usr/local/bin/
COPY --from=builder /usr/local/bin/bitcoin-cli /usr/local/bin/
COPY --from=builder /usr/local/bin/bitcoin-tx /usr/local/bin/
COPY --from=builder /usr/local/bin/bitcoin-util /usr/local/bin/
COPY --from=builder /usr/local/bin/bitcoin-wallet /usr/local/bin/
COPY --from=builder /usr/local/share/bitcoin.conf /usr/local/share/

# Create bitcoin user
RUN useradd -r bitcoin \
    && mkdir -p /home/bitcoin/.bitcoin \
    && chown -R bitcoin:bitcoin /home/bitcoin/.bitcoin

USER bitcoin
WORKDIR /home/bitcoin

# Expose mainnet ports
EXPOSE 8332 8333

# Default data directory
VOLUME ["/home/bitcoin/.bitcoin"]

CMD ["bitcoind", "-printtoconsole"]
