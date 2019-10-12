
FROM ubuntu:bionic

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root 'vscode' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install git iproute2 procps lsb-release \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME 

# Fetch N64 Toolchain
RUN apt-get -y install curl unzip \
    && curl -SL https://github.com/glankk/n64/archive/master.zip > /n64-master.zip \
    && pwd \
    && ls -la \
    && unzip /n64-master.zip

# Install N64 Toolchain
RUN cd /n64-master \
    && pwd \
    && ls -la \
    && chmod +x ./install_deps ./configure \
    && ./install_deps \
    && ./configure --prefix=/opt/n64 --enable-vc \
    && make install-toolchain \
    && make \
    && make install \
    && make install-sys

ENV PATH /opt/n64/bin:$PATH

# Fetch GZInject
RUN apt-get -y install curl unzip \
    && curl -SL https://github.com/krimtonz/gzinject/archive/master.zip > /gzinject-master.zip \
    && pwd \
    && ls -la \
    && unzip /gzinject-master.zip

# Install GZInject
RUN cd /gzinject-master \
    && pwd \
    && ls -la \
    && chmod +x ./configure \
    && ./configure --prefix=/opt/gzinject \
    && make \
    && make install

ENV PATH /opt/gzinject/bin:$PATH

# Clean up
RUN apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /n64-master \
    && rm -f /n64-master.zip \
    && rm -rf /gzinject-master \
    && rm -f /gzinject-master.zip

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=