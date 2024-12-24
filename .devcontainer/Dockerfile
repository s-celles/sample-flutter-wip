FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=daytona
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Update and install required packages
RUN apt-get update && apt-get install -y \
    git \
    curl \
    wget \
    unzip \
    xz-utils \
    libglu1-mesa \
    && rm -rf /var/lib/apt/lists/*

# Add non-root user to the image
RUN	groupadd --gid $USER_GID $USERNAME \
 	&& useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
 	&& echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
 	&& chmod 0440 /etc/sudoers.d/$USERNAME \
 	&& su $USERNAME \
 	&& cd $HOME

# Define variables
ARG FLUTTER_SDK=/usr/local/flutter
ARG FLUTTER_VERSION=3.27.1
ARG APP=/app/

# Git clone flutter repository
RUN git clone https://github.com/flutter/flutter.git $FLUTTER_SDK

# Change dir to current flutter folder and make a checkout to the specific version
RUN cd $FLUTTER_SDK && git fetch && git checkout $FLUTTER_VERSION

# Setup the flutter path as an enviromental variable
ENV PATH="$FLUTTER_SDK/bin:$FLUTTER_SDK/bin/cache/dart-sdk/bin:${PATH}"

# Start to run Flutter commands
# doctor to see if all was installes ok
RUN flutter doctor -v
