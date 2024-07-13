FROM nvidia/cuda:11.1.1-cudnn8-devel-ubuntu20.04

RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/lib/libcuda.so.1

ENV APT_INSTALL="apt-get install -y --no-install-recommends"
ENV DEBIAN_FRONTEND=noninteractive

RUN rm -rf /var/lib/apt/lists/* \
    /etc/apt/sources.list.d/cuda.list \
    /etc/apt/sources.list.d/nvidia-ml.list
RUN apt-get update 

RUN DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
    build-essential \
    ca-certificates \
    wget \
    git \
    g++ \
    cmake \
    # for MKL
    apt-transport-https gpg-agent gnupg2 \
    # for kenlm
    libboost-thread-dev libboost-test-dev libboost-system-dev libboost-program-options-dev \
    # for arrayfire
    libboost-stacktrace-dev \
    # FFTW
    libfftw3-dev \
    # ssh for OpenMPI
    openssh-server openssh-client \
    # for OpenMPI
    libopenmpi-dev openmpi-bin \
    # for kenlm
    zlib1g-dev libbz2-dev liblzma-dev && \
    apt-get clean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*

RUN cd /tmp && \
    git clone --branch v3.8.3 --depth 1 --recursive --shallow-submodules https://github.com/arrayfire/arrayfire.git && \
    mkdir -p arrayfire/build && cd arrayfire/build && \
    CXXFLAGS=-DOS_LNX cmake .. -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/arrayfire \
    -DAF_BUILD_CUDA=ON \
    -DAF_BUILD_CPU=OFF \
    -DAF_BUILD_OPENCL=OFF \
    -DAF_BUILD_EXAMPLES=OFF \
    -DAF_WITH_IMAGEIO=OFF \
    -DBUILD_TESTING=OFF \
    -DAF_BUILD_DOCS=OFF && \
    make install -j$(nproc)

RUN cd /tmp && git clone https://github.com/kpu/kenlm.git && \
    cd kenlm && git checkout 9af679c38477b564c26917a5dcf52d2c86177fb9 && \
    mkdir build && cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/kenlm \
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON && \
    make install -j$(nproc)
ENV KENLM_ROOT=/opt/kenlm

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
    vim \
    emacs \
    nano \
    htop \
    curl \ 
    mc \
    zsh \
    # nccl: for flashlight
    libnccl2 libnccl-dev \
    # libsndfile
    libsndfile1-dev \
    # for Intel's Math Kernel Library (MKL)
    cpio \
    # gflags
    libgflags-dev libgflags2.2 \
    # for glog
    libgoogle-glog-dev libgoogle-glog0v5 \
    # python sox
    sox libsox-dev python3-dev python3-pip python3-distutils 
# python (for bindings ad preprocessing)

RUN python3 -m pip --no-cache-dir install --upgrade numpy typing-extensions
RUN python3 -m pip --no-cache-dir install --upgrade setuptools sox tqdm && \
    # ==================================================================
    # clean up everything
    # ------------------------------------------------------------------
    apt-get clean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*
# ==================================================================
# MKL https://software.intel.com/en-us/mkl
# ------------------------------------------------------------------

#RUN echo 'Acquire::http::Proxy "http://192.168.230.150:8123";' > /etc/apt/apt.conf

RUN wget -e http-proxy=192.168.230.150:8123 -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
    | gpg --dearmor | tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null

RUN echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | tee /etc/apt/sources.list.d/oneAPI.list
RUN apt update -y && apt install intel-oneapi-mkl-devel intel-oneapi-dnnl-devel -y && \
    # ==================================================================
    # clean up everything
    # ------------------------------------------------------------------
    apt-get clean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/*

RUN git clone https://github.com/USCiLab/cereal && \
    cd cereal && \
    mkdir -p build && cd build && \
    cmake .. && make install -j8 

#ENV MKL_ROOT=/opt/intel/oneapi/mkl/latest

# ==================================================================
# flashlight with CUDA backend
# ------------------------------------------------------------------
# Setup and build flashlight
WORKDIR /

RUN git clone https://github.com/flashlight/flashlight.git -b 0.3

RUN . /opt/intel/oneapi/setvars.sh &&\
    cd /flashlight && mkdir -p build && \
    cd build && cmake .. -DFL_BUILD_APP_ASR=ON \
    -DFL_BUILD_PKG_SPEECH=ON \
    -DFL_BUILD_LIB_COMMON=ON \
    -DFL_BUILD_LIB_AUDIO=ON \
    -DFL_BUILD_LIB_TEXT=ON \
    -DFL_BUILD_LIB_SEQUENCE=ON \
    -DFL_BUILD_PKG_RUNTIME=ON \
    -DFL_BUILD_TESTS=ON \
    -DFL_BUILD_CONTRIB=ON \
    -DFL_USE_ONEDNN=OFF \
    -DFL_BUILD_EXAMPLES=ON \
    -DCMAKE_INSTALL_PREFIX=/opt/flashlight \
    -DFL_BACKEND=CUDA \
    -DCMAKE_BUILD_TYPE=Release

RUN . /opt/intel/oneapi/setvars.sh && \
    cd /flashlight/build && \
    make install
    
RUN echo "\nsource /opt/intel/oneapi/setvars.sh\n" >> /root/.bashrc 
# Install Oh My Zsh
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
RUN echo "\nif test -t 1; then exec zsh; fi\n" >> /root/.bashrc
