# Docker for proyect glozzom (BASE)
# ros-melodic-base, gazebo9, gps, python libraries 
# Python 3 version

ARG UBUNTU_VERSION=18.04

FROM tensorflow/tensorflow:latest-gpu-py3
LABEL maintainer="Wang, Yan" email="wangyan@frtech.fr"

#### Use an official ROS runtime as a parent image

# 
RUN apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/3bf863cc.pub
# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
    ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    apt-get update && apt-get install -q -y tzdata 

# setup sources.list
RUN sh -c '. /etc/lsb-release && echo "deb http://mirrors.ustc.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'

# setup keys
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# install bootstrap tools
RUN apt-get update && apt-get install --no-install-recommends -y \
    python-rosdep \
    python-rosinstall \
    python-vcstools    

# setup environment
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

# bootstrap rosdep
COPY ros_bypass/hosts /etc/hosts
COPY ros_bypass/resolv.conf /etc/resolv.conf
COPY ros_bypass/ros_github /etc/ros/ros_github
RUN rosdep init

COPY ros_bypass/rosdep_bypass/__init__.py /usr/lib/python2.7/dist-packages/rosdistro/__init__.py
COPY ros_bypass/rosdep_bypass/20-default.list /etc/ros/rosdep/sources.list.d/20-default.list
RUN rosdep update

# install ros packages
ENV ROS_DISTRO melodic
RUN apt-get update && apt-get install -y \
    ros-melodic-ros-core


# install ros packages
RUN apt-get install -y \
    ros-melodic-ros-base


# install universal robot ros packages
RUN apt-get install -y \
    ros-$ROS_DISTRO-gazebo-ros-pkgs \
    ros-$ROS_DISTRO-rqt-common-plugins \
    # install catkin
    ros-$ROS_DISTRO-catkin \
    python-catkin-tools \
    # Install Numpy Boost
    libboost-dev \
    libboost-python-dev \
    libboost-system-dev \
    libboost-all-dev \
    libatlas-base-dev \
    libprotobuf-dev \
    protobuf-compiler \
    # python dependencies
    python-setuptools \
    python3-tk \
    python3-numpy \
    # utils
    locate \
    aptitude \
    vim htop \
    curl wget \
    tk \
    spacenavd


## Python libraries ##
RUN python -m pip install pip --upgrade && \
    pip install \
    matplotlib==2.2.3 \
    spicy \
    protobuf \
    pyyaml \
    pyquaternion \
    rospkg \
    lxml \
    tqdm \
    catkin-pkg \
    empy \
    PyVirtualDisplay \
    defusedxml \
    gym \
    psutil \
    pyprind

### Gazebo ###
# setup keys
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys D2486D2DD83DB69272AFE98867170598AF249743

# setup sources.list
RUN . /etc/os-release \
    && echo "deb http://packages.osrfoundation.org/gazebo/$ID-stable `lsb_release -sc` main" > /etc/apt/sources.list.d/gazebo-latest.list

ENV DEBIAN_FRONTEND noninteractive

# install gazebo packages
RUN apt-get update && apt-get install -q -y \
    binutils \
    mesa-utils \
    module-init-tools \
    x-window-system \
    gazebo9 \
    libgazebo9-dev


ADD docker/orocos_kinematics_dynamics.tar.xz /root/

# Install SIP 4.19.8
ADD docker/sip-4.19.8.tar.gz /root/

RUN /bin/bash -c "cd ~/ \
    && cd sip-4.19.8 \
    && python configure.py \
    && make -j4 && make install"

# Install PyKDL
RUN apt update && apt -y install libeigen3-dev 
RUN /bin/bash -c "cd ~/orocos_kinematics_dynamics/orocos_kdl \
    && mkdir build && cd build \
    && cmake -DCMAKE_BUILD_TYPE=Release .. \
    && make -j4 && make install"

RUN /bin/bash -c "cd ~/orocos_kinematics_dynamics/python_orocos_kdl \
    && mkdir build && cd build \
    && cmake -DCMAKE_BUILD_TYPE=Release -DPYTHON_INCLUDE_DIR=$(python -c "from distutils.sysconfig import get_python_inc; print(get_python_inc())")  -DPYTHON_LIBRARY=$(python -c "import distutils.sysconfig as sysconfig; print(sysconfig.get_config_var('LIBDIR'))") -DPYTHON_VERSION=3 .. \
    && make -j4"

COPY docker/bashrc /etc/bash.bashrc
RUN chmod a+rwx /etc/bash.bashrc

ENV PYTHONIOENCODING UTF-8

# add rqt-multiplot
RUN apt-get update && apt-get install -y ros-$ROS_DISTRO-rqt-multiplot

RUN pip install xmltodict imageio IPython

RUN pip install PySide2 PyQt5 transform3d

RUN pip3 install pyserial

################################################
# Custom python libs
################################################

RUN pip install Cython cpprb

RUN apt install -y python-pip
#RUN python2 -m pip install pyquaternion scipy Cython

# fix for sip
RUN rm /usr/lib/python3/dist-packages/sip.cpython-36m-x86_64-linux-gnu.so
# fix gazebo
RUN apt update && apt-mark hold cuda-compat-10-1 libnvinfer-plugin6 libnvinfer6 libcudnn7 && apt upgrade -y 

RUN pip install future netifaces pandas sklearn seaborn

COPY docker/gazebo-gui.ini /root/.gazebo/gui.ini

# fix TF for python3
RUN pip3 install --extra-index-url https://rospypi.github.io/simple/ tf2_ros
# fix gym
RUN pip install gym[atari]==0.15.7 -i https://mirrors.bfsu.edu.cn/pypi/web/simple/

### End ###

# clean
RUN pip cache purge
RUN rm -rf /var/lib/apt/lists/*

# setup environment
EXPOSE 11345
ENV NVIDIA_VISIBLE_DEVICES ${NVIDIA_VISIBLE_DEVICES:-all}
ENV NVIDIA_DRIVER_CAPABILITIES ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics

RUN echo 'source /opt/ros/$ROS_DISTRO/setup.bash' >> ~/.bashrc

RUN echo 'source /usr/share/gazebo/setup.sh' >> ~/.bashrc
RUN echo 'export PYTHONPATH=/root/orocos_kinematics_dynamics/python_orocos_kdl/build:/root/gps:$PYTHONPATH' >> ~/.bashrc

