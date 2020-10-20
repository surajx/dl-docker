FROM ubuntu:20.04

LABEL maintainer="Suraj Narayanan Sasikumar suraj@hessianailabs.com"

# Needed for string substitution
SHELL ["/bin/bash", "-c"]

ENV TZ=Australia/Sydney
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && echo ${TZ} > /etc/timezone


RUN apt-get update && apt-get install -y --no-install-recommends \
    gnupg2 curl ca-certificates && \
    curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub | apt-key add - && \
    echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list && \
    echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/nvidia-ml.list \
    && rm -rf /var/lib/apt/lists/*

ENV CUDA_VERSION 10.1.243
ENV CUDA_PKG_VERSION 10-1=$CUDA_VERSION-1
ENV CUDNN_VERSION 8.0.4.30
ENV LIBNVINFER 6.0.1-1
ENV NCCL_VERSION 2.7.8

ARG CUDA_MAJOR_MINOR_VERSION=10.1
ARG CUDNN_MAJOR_VERSION=8
ARG LIBNVINFER_MAJOR_VERSION=6

LABEL com.nvidia.cudnn.version="${CUDNN_VERSION}"

# For libraries in the cuda-compat-* package: https://docs.nvidia.com/cuda/eula/index.html#attachment-a
RUN apt-get update && apt-get install -y --no-install-recommends \
    cuda-cudart-$CUDA_PKG_VERSION \
    cuda-compat-10-1 \
    cuda-nvrtc-$CUDA_PKG_VERSION \
    cuda-cufft-$CUDA_PKG_VERSION \
    cuda-curand-$CUDA_PKG_VERSION \
    cuda-cusolver-$CUDA_PKG_VERSION \
    cuda-cusparse-$CUDA_PKG_VERSION \
    cuda-libraries-$CUDA_PKG_VERSION \
    cuda-npp-$CUDA_PKG_VERSION \
    cuda-nvtx-$CUDA_PKG_VERSION \
    # There appears to be a regression in libcublas10=10.2.2.89-1 which
    # prevents cublas from initializing in TF. See
    # https://github.com/tensorflow/tensorflow/issues/9489#issuecomment-562394257    
    libcublas10=10.2.1.243-1 \
    libcublas-dev=10.2.1.243-1 \
    libnccl2=$NCCL_VERSION-1+cuda${CUDA_MAJOR_MINOR_VERSION} \
    libnccl-dev=$NCCL_VERSION-1+cuda${CUDA_MAJOR_MINOR_VERSION} \
    cuda-nvml-dev-$CUDA_PKG_VERSION \
    cuda-command-line-tools-$CUDA_PKG_VERSION \
    cuda-nvprof-$CUDA_PKG_VERSION \
    cuda-npp-dev-$CUDA_PKG_VERSION \
    cuda-libraries-dev-$CUDA_PKG_VERSION \
    cuda-minimal-build-$CUDA_PKG_VERSION \
    libcudnn${CUDNN_MAJOR_VERSION}=$CUDNN_VERSION-1+cuda${CUDA_MAJOR_MINOR_VERSION} \
    libcudnn${CUDNN_MAJOR_VERSION}-dev=$CUDNN_VERSION-1+cuda${CUDA_MAJOR_MINOR_VERSION} \  
    # Install TensorRT 
    libnvinfer${LIBNVINFER_MAJOR_VERSION}=${LIBNVINFER}+cuda${CUDA_MAJOR_MINOR_VERSION} \
    libnvinfer-plugin${LIBNVINFER_MAJOR_VERSION}=${LIBNVINFER}+cuda${CUDA_MAJOR_MINOR_VERSION} \
    libnvinfer-dev=${LIBNVINFER}+cuda${CUDA_MAJOR_MINOR_VERSION} \      
    && ln -s cuda-10.1 /usr/local/cuda && \
    rm -rf /var/lib/apt/lists/*

# Required for nvidia-docker v1
RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf && \
    echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

ENV PATH /usr/local/nvidia/bin:/usr/local/cuda/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64
ENV LIBRARY_PATH /usr/local/cuda/lib64/stubs

# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
ENV NVIDIA_REQUIRE_CUDA "cuda>=10.1 brand=tesla,driver>=396,driver<397 brand=tesla,driver>=410,driver<411 brand=tesla,driver>=418,driver<419"

# apt from auto upgrading the cublas, nccl2 package. See https://gitlab.com/nvidia/container-images/cuda/-/issues/88
RUN apt-mark hold \
    libcublas10 \
    libnccl2 \
    libcublas-dev \
    libcudnn${CUDNN_MAJOR_VERSION} \
    libcudnn${CUDNN_MAJOR_VERSION}-dev \
    libnccl-dev

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    build-essential \
    bzr \
    apt-utils \
    wget \
    net-tools \
    libfreetype6-dev \
    libhdf5-serial-dev \
    libzmq3-dev \
    pkg-config \
    software-properties-common \
    unzip \
    libx11-dev \
    libxkbfile-dev \
    libsecret-1-dev \
    git \
    jq \
    vim \
    sudo \
    supervisor \
    binutils \
    nginx \
    lua5.1 \
    axel \
    imagemagick \
    openssh-client \
    g++ \
    gcc \
    bash-completion \
    cmake \
    iptables \
    golang \
    htop \
    python3-pip \
    procps \
    autoconf \
    automake \
    bzip2 \
    file \
    make \
    # Cant install nvtop from apt, see: https://github.com/Syllo/nvtop/issues/73
    # nvtop \
    graphviz \
    rsync \
    protobuf-compiler \
    openmpi-bin \
    dpkg-dev \
    tcl \
    tk \
    tcl-dev \
    tk-dev \
    nodejs \
    npm \
    swig \
    python3-pip \
    python3-dev \
    #Cleanup
    && apt-get auto-remove \
    && apt-get clean \    
    && rm -rf /var/lib/apt/lists/*

# For CUDA profiling, TensorFlow requires CUPTI.
ENV LD_LIBRARY_PATH /usr/local/cuda/extras/CUPTI/lib64:/usr/local/cuda/lib64:$LD_LIBRARY_PATH

RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 \
    && echo "/usr/local/cuda/lib64/stubs" > /etc/ld.so.conf.d/z-cuda-stubs.conf \
    && ldconfig

RUN python3 -m pip install --no-cache-dir -U \
    pip \
    setuptools

RUN ln -s $(which python3) /usr/local/bin/python

RUN python3 -m pip install --no-cache-dir -U \
    #TF
    tensorflow\
    #Jupyter
    jupyter \    
    jupyter_http_over_ws \
    ipykernel \
    nbformat \
    jupyterlab \  
    #TF and DL/RL Extras  
    tensorflow-probability \
    tensorflow-addons \
    tensorflow-hub \
    tensorflow-datasets \
    gym[atari,box2d,classic_control] \
    tf-agents[reverb] \
    dm-sonnet \
    dm-control \
    bsuite \
    gym-retro \
    retrowrapper \    
    #ML + Viz
    transformers \
    matplotlib \
    nltk \
    scikit-learn \
    scikit-image \
    scikit-multiflow\
    scipy \
    kaggle \
    seaborn \    
    plotly \
    pydot \
    pandas \
    plotnine \
    h5py \
    annoy \
    # Extras
    flask \
    fastapi \
    pyvirtualdisplay \
    Pillow \
    incremental \
    tqdm \
    six \
    imageio \
    && rm -rf /tmp/* /var/tmp/*

#PyTorch
RUN python3 -m pip install --no-cache-dir\
    torch==1.6.0+cu101 \
    torchvision==0.7.0+cu101 \
    -f https://download.pytorch.org/whl/torch_stable.html \
    && rm -rf /tmp/* /var/tmp/*

RUN jupyter serverextension enable --py jupyter_http_over_ws

COPY bashrc /etc/bash.bashrc
RUN chmod 644 /etc/bash.bashrc

EXPOSE 8888
