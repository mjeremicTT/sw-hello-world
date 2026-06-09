# TT-UMD UBUNTU 22.04 AMD64 DOCKERFILE
# Based on tenstorrent/tt-umd/.github/ubuntu-22.04.Dockerfile
FROM ubuntu:22.04

ARG ARCH_NAME
ENV ARCH_NAME=${ARCH_NAME}
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
RUN wget https://apt.llvm.org/llvm.sh \
    && chmod u+x llvm.sh \
    && ./llvm.sh 20 \
    && apt-get install -y --no-install-recommends libc++-20-dev libc++abi-20-dev \
    && ln -sf /usr/bin/clang-20 /usr/bin/clang \
    && ln -sf /usr/bin/clang++-20 /usr/bin/clang++ \
    && rm -rf /var/lib/apt/lists/* llvm.sh

RUN git clone --recurse-submodules https://github.com/tenstorrent/tt-umd /usr/tt-umd

WORKDIR /usr/tt-umd

ENV CC=clang-20
ENV CXX=clang++-20

RUN cmake -B build -G Ninja -DTT_UMD_BUILD_TESTS=ON -DBUILD_SHARED_LIBS=OFF \
    && cmake --build build --target umd_tests

RUN ln -s build/test/umd/${ARCH_NAME}/unit_tests run_unit_tests && chmod ugo+x run_unit_tests

CMD ["./run_unit_tests"]
