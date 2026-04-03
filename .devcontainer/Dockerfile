FROM mcr.microsoft.com/devcontainers/base:ubuntu

RUN apt-get update && apt-get install -y \
    wget \
    tar \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

RUN wget -q https://github.com/YosysHQ/oss-cad-suite-build/releases/download/2026-03-07/oss-cad-suite-linux-x64-20260307.tgz \
    && tar xzf oss-cad-suite-linux-x64-20260307.tgz \
    && rm oss-cad-suite-linux-x64-20260307.tgz

ENV PATH="/opt/oss-cad-suite/bin:${PATH}"