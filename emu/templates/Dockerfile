# Copyright 2019 - The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
FROM debian:stretch-slim

# Install all the required emulator dependencies.
# You can get these by running ./android/scripts/unix/run_tests.sh --verbose --verbose --debs | grep apt | sort -u
# pulse audio is needed due to some webrtc dependencies.
RUN apt-get update && apt-get install -y \
# Needed for install
    unzip \
# Emulator & video bridge dependencies
    libc6 libdbus-1-3 libfontconfig1 libgcc1 \
    libpulse0 libtinfo5 libx11-6 libxcb1 libxdamage1 \
    libnss3 libxcomposite1 libxcursor1 \
    libxext6 libxfixes3 zlib1g libgl1 pulseaudio socat

# Next we get an android image ready
# We explicitly curl the image from a public site to make sure we
# don't accidentally publish internal testing images.
# Now we configure the user account under which we will be running the emulator
RUN mkdir -p /android/sdk/platforms && \
    mkdir -p /android/sdk/platform-tools && \
    mkdir -p /android/sdk/system-images && \
    mkdir -p /android-home

# Make sure to place files that do not change often in the higher layers
# as this will improve caching.
COPY launch-emulator.sh /android/sdk/
COPY default.pa /android/sdk/
COPY platform-tools /android/sdk/
COPY default.pa /etc/pulse/default.pa

RUN gpasswd -a root audio && \
    chmod +x /android/sdk/launch-emulator.sh

COPY {{emu_zip}} /android/sdk/
RUN unzip -u -o /android/sdk/{{emu_zip}} -d /android/sdk/ && \
    rm /android/sdk/{{emu_zip}}

COPY {{sysimg_zip}} /android/sdk/
RUN unzip -u -o /android/sdk/{{sysimg_zip}} -d /android/sdk/system-images/android && \
    rm /android/sdk/{{sysimg_zip}}

COPY avd /android-home
# Create an initial snapshot so we will boot fast next time around,
# This is currently an experimental feature, and is not easily configurable//
# RUN --security=insecure cd /android/sdk && ./launch-emulator.sh -quit-after-boot 120

# Open up adb & grpc port
EXPOSE 5555
EXPOSE 5556
ENV ANDROID_SDK_ROOT /android/sdk
ENV ANDROID_AVD_HOME /android-home
WORKDIR /android/sdk

# Note, we will not be using gpu acceleration, nor will the emulator be visible.
# You will need to make use of the grpc snapshot/webrtc functionality to actually interact with
# the emulator.
CMD ["/android/sdk/launch-emulator.sh"]

# Date frequently changes, so we place this in the last layer.
LABEL maintainer="{{user}}" \
      com.google.android.emulator.build-date="{{date}}" \
      com.google.android.emulator.description="Pixel 2 Emulator, running API {{api}}" \
      com.google.android.emulator.version="{{tag}}-{{api}}-{{abi}}/{{emu_build_id}}"
