ARG IMAGE=arm64v8/alpine:3.15

FROM $IMAGE

RUN  apk add zsh git curl wget g++ clang boost-static ninja boost-dev cmake make sudo bash vim libressl-dev util-linux-dev zlib-dev zlib-static

RUN mkdir /buildenv

WORKDIR /buildenv

RUN curl -JO 'https://developer.apple.com/file/?file=security&agree=Yes' -H 'Referer: https://developer.apple.com/security/' && unzip corecrypto.zip && rm -rf __MACOSX && (test -d corecrypto || mv corecrypto-*/ corecrypto)
# Apple's zip references scripts/code-coverage.cmake but doesn't ship it - it's an
# external helper from https://github.com/StableCoder/cmake-scripts that CMakeLists.txt
# assumes is present (normally added via a submodule/build step not included in the zip).
RUN mkdir -p /buildenv/corecrypto/scripts && curl -sL -o /buildenv/corecrypto/scripts/code-coverage.cmake "https://raw.githubusercontent.com/StableCoder/cmake-scripts/main/code-coverage.cmake"
# CoreCryptoSources.cmake references "corecrypto_static/ccrng_static.c" but the public zip
# ships the file flat at the top level - symlink it into place under the expected path.
RUN mkdir -p /buildenv/corecrypto/corecrypto_static && ln -s ../ccrng_static.c /buildenv/corecrypto/corecrypto_static/ccrng_static.c
WORKDIR /buildenv/corecrypto
RUN mkdir build; cd build; CC=clang CXX=clang++ cmake ..;
WORKDIR /buildenv/corecrypto/build
RUN sed -i -E 's|^(all: CMakeFiles\/corecrypto_perf)|#\1|' CMakeFiles/Makefile2; sed -i -E 's|^(all: CMakeFiles\/corecrypto_test)|#\1|' CMakeFiles/Makefile2
RUN make -j6; make install

WORKDIR /buildenv

RUN git clone --recursive https://github.com/microsoft/cpprestsdk; 
WORKDIR /buildenv/cpprestsdk
RUN sed -i 's|-Wcast-align||' "./Release/CMakeLists.txt"
RUN mkdir build; cd build; cmake -DBUILD_SHARED_LIBS=0 ..; make -j6; make install

WORKDIR /buildenv/

RUN git clone https://github.com/nih-at/libzip && cd libzip; mkdir build; cd build; cmake -DBUILD_SHARED_LIBS=0 ..; make -j6; make install


