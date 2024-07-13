FROM ubuntu:22.04 AS cpu_base_builder

ENV APT_INSTALL="apt install -y --no-install-recommends"

RUN apt update && \
    $APT_INSTALL \
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
    # for arrayfire CPU backend
    libboost-stacktrace-dev \
    # OpenBLAS
    libopenblas-dev liblapacke-dev \
    # ATLAS
    libatlas3-base libatlas-base-dev liblapacke-dev \
    # FFTW
    libfftw3-dev \
    # ssh for OpenMPI
    openssh-server openssh-client \
    # for OpenMPI
    libopenmpi-dev openmpi-bin \
    # for kenlm
    zlib1g-dev libbz2-dev liblzma-dev && \
    # ==================================================================
    # clean up everything
    # ------------------------------------------------------------------
    apt-get clean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*

#############################################################################
#                                DEPS IMAGES                                #
#############################################################################

FROM cpu_base_builder AS cpu_gloo
# ==================================================================
# Gloo https://github.com/facebookincubator/gloo.git
# ------------------------------------------------------------------
RUN cd /tmp && git clone https://github.com/facebookincubator/gloo.git && \
    cd gloo && git checkout 56b221c0a811491d2dc2a3254b468ad687bbdaab && mkdir build && cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/gloo \
    -DUSE_MPI=ON && \
    make install -j$(nproc)

FROM cpu_base_builder AS cpu_kenlm
# ==================================================================
# KenLM https://github.com/kpu/kenlm
# ------------------------------------------------------------------
RUN cd /tmp && git clone https://github.com/kpu/kenlm.git && \
    cd kenlm && git checkout 9af679c38477b564c26917a5dcf52d2c86177fb9 && \
    mkdir build && cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/kenlm \
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON && \
    make install -j$(nproc)

#############################################################################
#                             FINAL IMAGE                                   #
#############################################################################

FROM cpu_base_builder AS cpu_base_final

# ==================================================================
# arrayfire with CPU backend https://github.com/arrayfire/arrayfire/wiki/
# ------------------------------------------------------------------
RUN apt update && apt install ocl-icd-opencl-dev -y

RUN cd /tmp && git clone --branch v3.8.3 --depth 1 --recursive --shallow-submodules https://github.com/arrayfire/arrayfire.git && \
    mkdir -p arrayfire/build && cd arrayfire/build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release \
    -DAF_BUILD_CPU=OFF \
    -DAF_BUILD_CUDA=OFF \
    -DAF_BUILD_OPENCL=ON \
    -DAF_BUILD_EXAMPLES=OFF \
    -DAF_WITH_IMAGEIO=OFF \
    -DBUILD_TESTING=OFF \
    -DAF_BUILD_DOCS=OFF && \
    make install -j$(nproc)

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
    vim \
    emacs \
    nano \
    htop \
    # libsndfile
    libsndfile1-dev \
    # gflags
    libgflags-dev libgflags2.2 \
    # for glog
    libgoogle-glog-dev libgoogle-glog0v5 \
    # python sox
    sox libsox-dev python3-dev python3-pip python3-distutils

# python (for bindings and preprocessing)
RUN python3 -m pip install numpy typing_extensions
RUN python3 -m pip install setuptools sox tqdm
# ==================================================================
# clean up everything
# ------------------------------------------------------------------
RUN apt clean && \
    apt -y autoremove && \
    rm -rf /var/lib/apt/lists/*
# ==================================================================
# MKL https://software.intel.com/en-us/mkl
# ------------------------------------------------------------------
RUN wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
    | gpg --dearmor | tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null

RUN echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | tee /etc/apt/sources.list.d/oneAPI.list
RUN apt update -y && apt install intel-oneapi-mkl-devel -y && \
    # ==================================================================
    # clean up everything
    # ------------------------------------------------------------------
    apt-get clean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/*


#COPY --from=cpu_onednn     /opt/onednn     /opt/onednn
COPY --from=cpu_gloo       /opt/gloo       /opt/gloo
COPY --from=cpu_kenlm      /opt/kenlm      /opt/kenlm

ENV MKLROOT="/opt/intel/mkl"
ENV KENLM_ROOT="/opt/kenlm"

WORKDIR /

RUN git clone https://github.com/flashlight/flashlight.git --branch v0.3.2

RUN . /opt/intel/oneapi/setvars.sh && cd /flashlight && mkdir -p build && \
    cd build && cmake .. -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/flashlight \
    -DFL_BUILD_APP_ASR=ON \
    -DFL_BUILD_PKG_SPEECH=ON \
    -DFL_BUILD_LIB_COMMON=ON \
    -DFL_BUILD_LIB_AUDIO=ON \
    -DFL_BUILD_LIB_TEXT=ON \
    -DFL_BUILD_LIB_SEQUENCE=ON \
    -DFL_BUILD_PKG_RUNTIME=ON \
    -DFL_BUILD_TESTS=ON \
    -DFL_BUILD_CONTRIB=ON \
    -DFL_BACKEND=OPENCL \
    -DGloo_DIR=/opt/gloo/share/cmake &&\
    make install -j$(nproc)

# ==================================================================
# Change Bashrc and Installing zsh
# ------------------------------------------------------------------
RUN echo "\nsource /opt/intel/oneapi/setvars.sh\n" >> /root/.bashrc 
RUN echo "\nif test -t 1; then exec zsh; fi\n" >> /root/.bashrc
# Install Oh My Zsh
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
