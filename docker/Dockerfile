ARG BCFTOOLS_VERSION=1.23.1
ARG HTSLIB_VERSION=1.23.1

FROM debian:12-slim AS builder

ENV DEBIAN_FRONTEND=noninteractive
ARG BCFTOOLS_VERSION
ARG HTSLIB_VERSION
ENV LD_LIBRARY_PATH=/opt/bcftools/lib
ENV BCFTOOLS_PLUGINS=/opt/bcftools/libexec/bcftools

RUN set -eux; \
    for attempt in 1 2 3; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            build-essential \
            ca-certificates \
            curl \
            libbz2-dev \
            libcurl4-gnutls-dev \
            libdeflate-dev \
            libgsl-dev \
            liblzma-dev \
            libssl-dev \
            zlib1g-dev; then \
        break; \
      fi; \
      if [ "$attempt" = "3" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/build

RUN curl -fsSL "https://github.com/samtools/htslib/releases/download/${HTSLIB_VERSION}/htslib-${HTSLIB_VERSION}.tar.bz2" \
      -o htslib.tar.bz2 \
 && tar -xjf htslib.tar.bz2 \
 && curl -fsSL "https://github.com/samtools/bcftools/releases/download/${BCFTOOLS_VERSION}/bcftools-${BCFTOOLS_VERSION}.tar.bz2" \
      -o bcftools.tar.bz2 \
 && tar -xjf bcftools.tar.bz2

WORKDIR /tmp/build/htslib-${HTSLIB_VERSION}

RUN ./configure --prefix=/opt/bcftools \
 && make -j"$(nproc)" \
 && make install

WORKDIR /tmp/build/bcftools-${BCFTOOLS_VERSION}

RUN ./configure \
      --prefix=/opt/bcftools \
      --with-htslib=/opt/bcftools \
      --with-bcf-plugin-dir=/opt/bcftools/libexec/bcftools \
      --with-bcf-plugin-path=/opt/bcftools/libexec/bcftools \
      --enable-libgsl \
 && make -j"$(nproc)" \
 && make install \
 && /opt/bcftools/bin/bcftools --version 2>&1 | grep -F "bcftools ${BCFTOOLS_VERSION}" >/dev/null \
 && /opt/bcftools/bin/bcftools polysomy -h 2>&1 | grep -F "bcftools polysomy" >/dev/null \
 && /opt/bcftools/bin/bcftools cnv -h 2>&1 | grep -F "bcftools cnv" >/dev/null \
 && /opt/bcftools/bin/bcftools plugin -l >/dev/null \
 && /opt/bcftools/bin/bgzip --help >/dev/null \
 && /opt/bcftools/bin/tabix --help >/dev/null

FROM debian:12-slim

ENV DEBIAN_FRONTEND=noninteractive
ARG BCFTOOLS_VERSION

RUN set -eux; \
    for attempt in 1 2 3; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            ca-certificates \
            libbz2-1.0 \
            libcurl3-gnutls \
            libdeflate0 \
            libgsl27 \
            liblzma5 \
            libssl3 \
            perl \
            python3 \
            python3-venv \
            zlib1g; then \
        break; \
      fi; \
      if [ "$attempt" = "3" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

RUN python3 -m venv /opt/python \
 && /opt/python/bin/pip install --no-cache-dir --upgrade pip setuptools wheel \
 && /opt/python/bin/pip install --no-cache-dir \
      gffutils==0.13 \
      numpy==1.26.4 \
      matplotlib==3.8.4

COPY --from=builder /opt/bcftools /opt/bcftools

ENV PATH=/opt/python/bin:/opt/bcftools/bin:${PATH}
ENV LD_LIBRARY_PATH=/opt/bcftools/lib
ENV BCFTOOLS_PLUGINS=/opt/bcftools/libexec/bcftools

RUN bcftools --help >/dev/null \
 && bcftools --version 2>&1 | grep -F "bcftools ${BCFTOOLS_VERSION}" >/dev/null \
 && bcftools polysomy -h 2>&1 | grep -F "bcftools polysomy" >/dev/null \
 && bcftools cnv -h 2>&1 | grep -F "bcftools cnv" >/dev/null \
 && bcftools plugin -l >/dev/null \
 && plot-vcfstats -h >/dev/null \
 && python -c "import gffutils, matplotlib, numpy" \
 && bgzip --help >/dev/null \
 && tabix --help >/dev/null

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=bcftools
