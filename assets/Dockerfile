FROM nvidia/cuda:11.3.1-cudnn8-devel-ubuntu20.04

ARG INSTALL_CW_PACKAGE=true
ARG CUDA_VERSION=11.3
ARG TORCH_VERSION=1.12.1
ARG CUDA_CHANNEL=cuda-11.3

ENV LANG=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
ENV LD_LIBRARY_PATH=/opt/.mujoco/mujoco200/bin:/usr/local/nvidia/lib:/usr/local/nvidia/lib64:${LD_LIBRARY_PATH}
ENV MUJOCO_PY_MUJOCO_PATH=/opt/.mujoco/mujoco200/
ENV MUJOCO_PY_MJKEY_PATH=/opt/.mujoco/mjkey.txt
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=all

RUN apt-get -y update -q
# Add deadsnakes PPA for Python 3.7
RUN apt-get install -y software-properties-common
RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt-get update

# Install Python 3.7 and other required packages
RUN apt-get -y install git ffmpeg python3.7 python3.7-dev python3.7-distutils curl libgl1-mesa-dev libgl1-mesa-glx libglew-dev \
               libosmesa6-dev net-tools unzip vim wget xpra xserver-xorg-dev git virtualenv \
               build-essential pkg-config nvidia-cuda-toolkit patchelf

# Update alternatives to use Python 3.7
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 1

# Install pip for Python 3.7 using the correct version of get-pip.py
RUN curl https://bootstrap.pypa.io/pip/3.7/get-pip.py -o get-pip.py && \
    python3 get-pip.py && \
    rm get-pip.py

RUN mkdir -p /opt/.mujoco \
    && wget https://www.roboti.us/download/mujoco200_linux.zip -O mujoco.zip \
    && unzip mujoco.zip -d /opt/.mujoco \
    && mv /opt/.mujoco/mujoco200_linux /opt/.mujoco/mujoco200 \
    && rm mujoco.zip

COPY mjkey.txt /opt/.mujoco/mjkey.txt

# Install specific versions of dependencies
RUN python3 -m pip install --no-cache-dir 'setuptools==57.5.0'
RUN python3 -m pip install --no-cache-dir 'Cython==0.29.21'
RUN python3 -m pip install --no-cache-dir 'numpy==1.21.6'

# Install older version of mujoco-py
RUN python3 -m pip install --no-cache-dir 'mujoco-py==2.0.2.8'

# Install Gym dependencies
RUN python3 -m pip install --no-cache-dir \
    'cloudpickle>=1.2.0' \
    'opencv-python>=3.0.0' \
    'pygame>=2.0.0' \
    'pyglet>=1.4.0' \
    'scipy>=1.4.1' \
    'box2d-py~=2.3.5' \
    'importlib-metadata<5.0'  # Add this for Python 3.7 compatibility

# Install specific version of gym
RUN python3 -m pip install --no-cache-dir 'gym==0.23.1'

# Install PyTorch with CUDA support
RUN python3 -m pip install --no-cache-dir torch==${TORCH_VERSION}+cu113 torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu113

RUN mkdir /continualworld
COPY continualworld/ /continualworld/continualworld
COPY examples/ /continualworld/examples
COPY /*.py /continualworld/
COPY /*.ipynb /continualworld/

RUN cd /continualworld && \
    if $INSTALL_CW_PACKAGE ; \
        then pip install -e . ; \
        else python3 setup.py egg_info && pip install -r continualworld.egg-info/requires.txt; \
    fi

RUN chmod a+rw -R /usr/local/lib/python3.7
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*
RUN chmod +r /opt/.mujoco/mjkey.txt

WORKDIR /continualworld