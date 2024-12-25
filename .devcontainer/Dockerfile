FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Define build arguments for versions
ARG FLUTTER_VERSION=3.27.1
ARG ANDROID_SDK_VERSION=33
ARG ANDROID_BUILD_TOOLS_VERSION=33.0.0
ARG ANDROID_STUDIO_VERSION=2023.1.1.26

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
    libgtk-3-common \
    libglu1-mesa \
    # Android Studio dependencies
    libc6:i386 \
    libncurses5:i386 \
    libstdc++6:i386 \
    lib32z1 \
    libbz2-1.0:i386 \
    && rm -rf /var/lib/apt/lists/*

# Install Chrome
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main" | tee /etc/apt/sources.list.d/google-chrome.list && \
    apt-get update && \
    apt-get install -y google-chrome-stable && \
    rm -rf /var/lib/apt/lists/*

# Set Chrome executable path
ENV CHROME_EXECUTABLE=/usr/bin/google-chrome

# Install Android Studio
RUN mkdir -p /opt/android-studio && \
    wget https://redirector.gvt1.com/edgedl/android/studio/ide-zips/${ANDROID_STUDIO_VERSION}/android-studio-${ANDROID_STUDIO_VERSION}-linux.tar.gz -O android-studio.tar.gz && \
    tar -xzf android-studio.tar.gz -C /opt && \
    rm android-studio.tar.gz && \
    ln -s /opt/android-studio/bin/studio.sh /usr/local/bin/android-studio

# Create and set up proper ownership for vscode user directories
RUN mkdir -p /home/vscode && \
    chown -R vscode:vscode /home/vscode && \
    chown -R vscode:vscode /opt/android-studio

# Define variables
ARG FLUTTER_SDK=/home/vscode/flutter
ARG APP=/app/

# Set up Android SDK in user's home directory
ENV ANDROID_SDK_ROOT=/home/vscode/android-sdk
ENV ANDROID_HOME=${ANDROID_SDK_ROOT}
RUN mkdir -p ${ANDROID_SDK_ROOT} && \
    chown -R vscode:vscode ${ANDROID_SDK_ROOT}
ENV PATH=${PATH}:${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin:${ANDROID_SDK_ROOT}/platform-tools:/opt/android-studio/bin

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

# Initialize Flutter and fix channel
RUN flutter precache && \
    flutter channel stable && \
    flutter upgrade

# Download Android Command line tools
RUN mkdir -p ${ANDROID_SDK_ROOT}/cmdline-tools && \
    wget https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip -O cmdline-tools.zip && \
    unzip cmdline-tools.zip && \
    mv cmdline-tools latest && \
    mv latest ${ANDROID_SDK_ROOT}/cmdline-tools/ && \
    rm cmdline-tools.zip

# Accept licenses and install Android build tools and platform tools
RUN yes | sdkmanager --licenses && \
    sdkmanager "build-tools;${ANDROID_BUILD_TOOLS_VERSION}" \
               "platform-tools" \
               "platforms;android-${ANDROID_SDK_VERSION}"

# Configure Android Studio
RUN mkdir -p /home/vscode/.AndroidStudio${ANDROID_STUDIO_VERSION} && \
    mkdir -p /home/vscode/.android && \
    echo '{"install": {"installed": true}}' > /home/vscode/.android/analytics.settings && \
    echo '{"disabled": true}' > /home/vscode/.android/analytics.settings

# Now run doctor to verify the complete setup
RUN flutter doctor -v

# Return to home directory
WORKDIR /home/vscode