# TT-UMD UBUNTU 22.04 AMD64 DOCKERFILE
# Based on tenstorrent/tt-umd/.github/ubuntu-22.04.Dockerfile
#
# umd_tests builds the unit tests of every architecture, so all layers up to the build are
# shared and ARCH_NAME only selects which unit tests run. Build one tag per architecture:
#   docker build --platform linux/amd64 -f tt-umd/Dockerfile \
#     --build-arg TT_UMD_REF=<tt-umd commit> --build-arg ARCH_NAME=<wormhole_b0|blackhole> \
#     -t ghcr.io/mjeremictt/sw-hello-world/tt-umd-hello-world:latest-<arch> .
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV OS_CODENAME=jammy

# Essentials needed for HTTPS + GPG
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    gnupg \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Kitware repo for cmake >= 3.25
RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null \
    | gpg --dearmor - | tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ ${OS_CODENAME} main" \
    | tee /etc/apt/sources.list.d/kitware.list >/dev/null

# Build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    software-properties-common \
    build-essential \
    cmake \
    ninja-build \
    git \
    git-lfs \
    pkg-config \
    libhwloc-dev \
    libgtest-dev \
    libyaml-cpp-dev \
    libboost-all-dev \
    python3-dev \
    patchelf \
    && rm -rf /var/lib/apt/lists/*

# Clang-20 (tt-umd's current default toolchain)
# llvm.sh probes apt.llvm.org with a HEAD request that fails, patched the same way as upstream
RUN wget -q -O llvm.sh https://apt.llvm.org/llvm.sh \
    && chmod u+x llvm.sh \
    && sed -i 's/wget -q --method=HEAD/wget -q --spider/' llvm.sh \
    && ./llvm.sh 20 -n ${OS_CODENAME} \
    && apt-get install -y --no-install-recommends libc++-20-dev libc++abi-20-dev \
    && ln -sf /usr/bin/clang-20 /usr/bin/clang \
    && ln -sf /usr/bin/clang++-20 /usr/bin/clang++ \
    && rm -rf /var/lib/apt/lists/* llvm.sh

# Pin a commit so a rebuild doesn't reuse a cached clone of an old main
ARG TT_UMD_REF=main
RUN git clone https://github.com/tenstorrent/tt-umd /usr/tt-umd \
    && git -C /usr/tt-umd checkout ${TT_UMD_REF} \
    && git -C /usr/tt-umd submodule update --init --recursive

WORKDIR /usr/tt-umd

ENV CC=clang-20
ENV CXX=clang++-20

# Same configuration as tt-umd CI (build-tests.yml); BUILD_JOBS caps parallelism on small builders
ARG BUILD_JOBS
RUN cmake --preset tests -DTT_UMD_ENABLE_UNITY_BUILD=ON -DCMAKE_BUILD_TYPE=Release \
    && cmake --build build --target umd_tests --parallel ${BUILD_JOBS}

ENV LD_LIBRARY_PATH=/usr/tt-umd/build/lib

ARG ARCH_NAME
ENV ARCH_NAME=${ARCH_NAME}

# Hardware suites from tt-umd CI (run-tests.yml), plus misc tests
CMD ["bash", "-c", "git log -1 --format='tt-umd %H %cd' && ./build/test/umd/api/api_tests && ./build/test/umd/${ARCH_NAME}/unit_tests && ./build/test/umd/test_pcie_device/test_pcie_device && ./build/test/umd/misc/umd_misc_tests && ./build/test/umd/unified/unified_tests"]
