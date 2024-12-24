FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Update and install required packages
RUN apt-get update && apt-get install -y \
    git \
    curl \
    wget \
    unzip \
    xz-utils \
    clang \
    cmake \
    ninja-build \
    pkg-config \
    openjdk-11-jdk \
    && rm -rf /var/lib/apt/lists/*

# Create and set up proper ownership for vscode user directories
RUN mkdir -p /home/vscode && \
    chown -R vscode:vscode /home/vscode

# Define variables
ARG FLUTTER_SDK=/home/vscode/flutter
ARG FLUTTER_VERSION=3.27.1
ARG APP=/app/

# Set up Android SDK
ENV ANDROID_SDK_ROOT=/usr/lib/android-sdk
ENV PATH=${PATH}:${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin:${ANDROID_SDK_ROOT}/platform-tools

# Switch to vscode user
USER vscode

# Git clone flutter repository
RUN git clone https://github.com/flutter/flutter.git $FLUTTER_SDK

# Setup the flutter path as an enviromental variable
ENV PATH="$FLUTTER_SDK/bin:$FLUTTER_SDK/bin/cache/dart-sdk/bin:${PATH}"

# Change dir to current flutter folder and make a checkout to the specific version
WORKDIR $FLUTTER_SDK
RUN git fetch && \
    git checkout ${FLUTTER_VERSION}

# Initialize Flutter (this will download necessary dependencies)
RUN flutter precache

# Now switch to stable channel
RUN flutter channel stable && \
    flutter doctor -v

# Download Android Command line tools
RUN mkdir -p ${ANDROID_SDK_ROOT}/cmdline-tools && \
    wget https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip -O cmdline-tools.zip && \
    unzip cmdline-tools.zip && \
    mv cmdline-tools latest && \
    mv latest ${ANDROID_SDK_ROOT}/cmdline-tools/ && \
    rm cmdline-tools.zip

# Accept licenses and install Android build tools and platform tools
RUN yes | sdkmanager --licenses && \
    sdkmanager "build-tools;33.0.0" "platform-tools" "platforms;android-33"

# Return to home directory
WORKDIR /home/vscode