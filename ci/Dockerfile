# Taichi Dockerfile for development
ARG UBUNTU
FROM nvidia/cuda:${UBUNTU}
ARG PYTHON
ARG TEST_OPTION
ARG PYPI_PWD
ENV PYPI_PWD=$PYPI_PWD
ENV DEBIAN_FRONTEND=noninteractive
LABEL maintainer="https://github.com/taichi-dev"

RUN apt-get update && \
    apt-get install -y software-properties-common \
                       $PYTHON \
                       python3-pip \
                       ${PYTHON}-dev\
                       libtinfo-dev \
                       clang-10 \
                       wget \
                       git \
                       libx11-dev \
                       libxrandr-dev \
                       libxinerama-dev \
                       libxcursor-dev \
                       libxi-dev \
                       libglu1-mesa-dev \
                       freeglut3-dev \
                       mesa-common-dev \
                       build-essential \
                       libssl-dev \
                       libidn11-dev \
                       libz-dev \
                       unzip

# Install the latest version of CMAKE v3.20.2 from source
WORKDIR /
RUN wget https://github.com/Kitware/CMake/releases/download/v3.20.5/cmake-3.20.5-linux-x86_64.tar.gz
RUN tar xf cmake-3.20.5-linux-x86_64.tar.gz
ENV PATH="/cmake-3.20.5-linux-x86_64/bin:$PATH"

# Intall LLVM 10
WORKDIR /
RUN wget https://github.com/taichi-dev/taichi_assets/releases/download/llvm10_linux/taichi-llvm-10.0.0-linux.zip
RUN unzip taichi-llvm-10.0.0-linux.zip
ENV PATH="/taichi-llvm-10.0.0-linux/bin:$PATH"

# Install Taichi from source
ENV CC="clang-10"
ENV CXX="clang++-10"
WORKDIR /taichi-dev

# Prevent docker caching when head changes
ADD https://api.github.com/repos/taichi-dev/taichi/git/refs/heads/master version.json
RUN git clone https://github.com/taichi-dev/taichi --depth=1 --branch=master
# Install Taichi's Python dependencies
RUN $PYTHON -m pip install --user -r taichi/requirements.txt
# Build Taichi from source
RUN cd taichi && \
    git submodule update --init --recursive --depth=1 && \
    mkdir build
WORKDIR /taichi-dev/taichi/build
RUN cmake .. -DPYTHON_EXECUTABLE=$PYTHON -DCUDA_VERSION='10.0' -DTI_WITH_CUDA:BOOL=True -DTI_WITH_CC:BOOL=ON -DCMAKE_CXX_COMPILER=$CXX
RUN make -j 8

# Link Taichi source repo to Python Path
ENV PATH="/taichi-dev/taichi/bin:$PATH"
ENV TAICHI_REPO_DIR="/taichi-dev/taichi/"
ENV PYTHONPATH="$TAICHI_REPO_DIR/python:$PYTHONPATH"
ENV LANG="C.UTF-8"

# Show ELF info
RUN ldd libtaichi_core.so
RUN strings libtaichi_core.so | grep GLIBC

# Install twine and upload project to pypi.
RUN $PYTHON -m pip install --user twine
WORKDIR /taichi-dev/taichi/python
RUN ti test -t 2
RUN $PYTHON build.py upload $TEST_OPTION
