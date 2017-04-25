FROM ubuntu:16.04
MAINTAINER Daniel Fernandez <dfmartinez@gmail.com>
# Based on https://github.com/phillipao/kodi-for-android-build-env
# by (Phillip Oertel <phillipao@gmail.com>)
#
# Environment for building Kodi for Android.
#
# Installs required packages and Android SDK/NDK. Currently intended more for
# documentation, and as a detailed spec of a kodi build environment. But it's
# also easy to use directly for doing Kodi development.

RUN apt-get update && apt-get install -y --no-install-recommends \
    autoconf \
    build-essential \
    cmake \
    curl \
    default-jdk \
    # Used by configure to check arch. Will incorrectly identify arch without it.
    file \
    gawk \
    git \
    gperf \
    lib32stdc++6 \
    lib32z1 \
    lib32z1-dev \
    libcurl4-openssl-dev \
    nasm \
    unzip \
    zip \
    zlib1g-dev \
    libasound2-dev \
    wget

WORKDIR /opt
RUN curl -O http://dl.google.com/android/android-sdk_r24.4.1-linux.tgz 
RUN tar xzf android-sdk_r24.4.1-linux.tgz 
RUN rm -f android-sdk_r24.4.1-linux.tgz

ENV ANDROID_HOME /opt/android-sdk-linux
ENV PATH ${PATH}:${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools

RUN echo y | android update sdk --filter android-21 --no-ui --force -a

ENV ANDROID_NDK_DIR android-ndk-r12b
ENV ANDROID_NDK_ZIP "${ANDROID_NDK_DIR}-linux-x86_64.zip"
ENV ANDROID_NDK_URL "https://dl.google.com/android/repository/${ANDROID_NDK_ZIP}"
RUN curl -O "${ANDROID_NDK_URL}"
RUN unzip "${ANDROID_NDK_ZIP}" -x "${ANDROID_NDK_DIR}/platforms/*"
RUN unzip "${ANDROID_NDK_ZIP}" "${ANDROID_NDK_DIR}/platforms/android-21/*"
RUN rm "${ANDROID_NDK_ZIP}"

RUN keytool -genkey -keystore ~/.android/debug.keystore -v -alias \
      androiddebugkey -dname "CN=Android Debug,O=Android,C=US" -keypass \
      android -storepass android -keyalg RSA -keysize 2048 -validity 10000

RUN echo y | android update sdk --all -u -t build-tools-20.0.0

WORKDIR /opt/${ANDROID_NDK_DIR}/build/tools
RUN ./make-standalone-toolchain.sh --ndk-dir=../.. \
      --install-dir=/opt/android-toolchain-arm/android-21 --platform=android-21 \
      --toolchain=arm-linux-androideabi-4.9

WORKDIR /root
RUN wget https://github.com/codesnake/libamcodec/archive/5e23a81.tar.gz
RUN tar xvf 5e23a81.tar.gz
COPY amcodec.patch /root
WORKDIR libamcodec-5e23a81b706f1748e5054d2cc60be92f84c0fb21
RUN patch -p1 < /root/amcodec.patch
RUN cp -va /usr/include/alsa /opt/android-toolchain-arm/android-21/sysroot/usr/include/
RUN CC=/opt/android-toolchain-arm/android-21/bin/arm-linux-androideabi-gcc SYSROOT_PREFIX=/opt/android-toolchain-arm/android-21/sysroot TARGET_PREFIX=/storage/src/kodi-depends/arm-linux-androideabi-android-21 PREFIX="$SYSROOT_PREFIX/usr" CROSS_PREFIX="$TARGET_PREFIX" make -C amavutils
RUN cp -va amavutils/*.so /opt/android-toolchain-arm/android-21/sysroot/usr/lib
RUN CC=/opt/android-toolchain-arm/android-21/bin/arm-linux-androideabi-gcc SYSROOT_PREFIX=/opt/android-toolchain-arm/android-21/sysroot TARGET_PREFIX=/storage/src/kodi-depends/arm-linux-androideabi-android-21 PREFIX="$SYSROOT_PREFIX/usr" CROSS_PREFIX="$TARGET_PREFIX" make -C amadec install
RUN CC=/opt/android-toolchain-arm/android-21/bin/arm-linux-androideabi-gcc SYSROOT_PREFIX=/opt/android-toolchain-arm/android-21/sysroot TARGET_PREFIX=/storage/src/kodi-depends/arm-linux-androideabi-android-21 PREFIX="$SYSROOT_PREFIX/usr" CROSS_PREFIX="$TARGET_PREFIX" make -C amcodec install


# The following steps don't really belong in a Dockerfile, because ideally you don't
# want data (i.e. the source code) inside a docker container. But I included them here
# to show how I used it. A more idiomatic docker setup would be to create containers
# around the configure and make commands, with the source mounted as a volume.
# ENV KODI_BRANCH "Krypton"
# WORKDIR /root
# RUN git clone -b "${KODI_BRANCH}" --single-branch git://github.com/xbmc/xbmc.git kodi-android
# COPY kodi_amcodec.patch /root
# WORKDIR /root/kodi-android
# RUN patch -p1 < /root/kodi_amcodec.patch
# WORKDIR /root/kodi-android/tools/depends

# RUN ./bootstrap
# RUN ./configure --with-tarballs=/opt/xbmc-depends/tarballs --host=arm-linux-androideabi \
#     --with-sdk-path=/opt/android-sdk-linux --with-ndk=/opt/${ANDROID_NDK_DIR} \
#     --with-toolchain=/opt/android-toolchain-arm/android-21 --prefix=/opt/android-dev/android/xbmc-depends

# These commands tend to fail, but they generate state as they do so. I
# think this project has some nondeterministic build failures, and that's why
# it's necessary to build several times. As such, I don't guarantee that the
# following commands will successfully build the project. But they worked for
# me at least once.
# RUN make -j20 || echo failed
# RUN make || echo failed
# RUN make -C target/libcdio-gplv3 distclean
# RUN make
# RUN make -j20 -C target/binary-addons

# COPY kodi_amcodec.patch /root
# WORKDIR /root/kodi-android
# RUN make -j20 -C tools/depends/target/xbmc

CMD ["/bin/bash"]
