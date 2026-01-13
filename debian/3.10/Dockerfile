FROM debian:12-slim AS compiler_with_deps_image

ARG GR_VERSION=3.10.12.0

# install build dependencies
RUN apt update && \
    apt install -y --no-install-recommends --no-install-suggests \
        wget git cmake build-essential nano \
        cppzmq-dev \
        libboost-all-dev \
        libcodec2-dev \
        libfftw3-dev \
        libgmp-dev \
        libgsl-dev \
        libgsm1-dev \
        liblog4cpp5-dev \
        libspdlog-dev \
        libvolk2-dev \
        libthrift-dev \
        libsdl1.2-dev \
        libsndfile1-dev \
        libzmq3-dev \
        libiio-dev \
        libad9361-dev \
        pybind11-dev \
        python3-click \
        python3-click-plugins \
        python3-dev \
        python3-gi \
        python3-gi-cairo \
        python3-lxml \
        python3-mako \
        python3-matplotlib \
        python3-numpy \
        python3-pybind11 \
        python3-setuptools \
        python3-sphinx \
        python3-yaml \
        python3-zmq \
    && rm -rf /var/lib/apt/lists/*

#python3-dev python3-numpy python3-zmq python3-packaging \
#pybind11-dev python3-pybind11 libgmp-dev \
#libboost-all-dev libzmq3-dev libgsl-dev libsdl1.2-dev libfftw3-dev \
#libgsm1-dev libcodec2-dev libcppunit-dev \

FROM compiler_with_deps_image AS compiling

WORKDIR /
RUN wget --no-check-certificate https://github.com/gnuradio/gnuradio/archive/refs/tags/v${GR_VERSION}.tar.gz
RUN tar xf v${GR_VERSION}.tar.gz

WORKDIR /gnuradio-${GR_VERSION}/
RUN mkdir build
RUN cmake -B build -DCMAKE_C_FLAGS="-Wno-deprecated-declarations" \
      -DCMAKE_CXX_FLAGS="-Wno-deprecated-declarations" \
      -DCMAKE_BUILD_TYPE=RelWithDebInfo . \
      -DENABLE_COMMON_PCH=OFF \
      -DENABLE_GR_AUDIO=OFF \
      -DENABLE_GR_BLOCKTOOL=OFF \
      -DENABLE_GR_IIO=OFF \
      -DENABLE_GR_MODTOOL=OFF \
      -DENABLE_GR_QTGUI=OFF \
      -DENABLE_GR_UHD=OFF \
      -DENABLE_GRC=OFF \
      -DENABLE_MANPAGES=OFF \
      -DENABLE_UHD_RFNOC=OFF

RUN cmake --build build --parallel $(($(nproc) + 2))
RUN cmake --install build/


FROM debian:12-slim AS runtime_image

# create virtual memory circular buffer (vmcircbuf) default config
RUN mkdir -p /root/.gnuradio/prefs
COPY <<EOF /root/.gnuradio/prefs/vmcircbuf_default_factory
gr::vmcircbuf_mmap_factory
EOF

# install runtime dependencies
RUN apt update && \
    apt install -y --no-install-recommends --no-install-suggests \
        cppzmq-dev \
        libad9361-0 \
        libboost-all-dev \
        libcodec2-1.0 \
        libfftw3-bin \
        libgmp10 \
        libgslcblas0 \
        libgsm1 \
        libiio0 \
        liblog4cpp5v5 \
        libsdl1.2debian \
        libsndfile1 \
        libspdlog1.10 \
        libthrift-0.17.0 \
        libvolk2-bin \
        libzmq5 \
        python3 \
        python3-click \
        python3-click-plugins \
        python3-mako \
        python3-matplotlib \
        python3-numpy \
        python3-pybind11 \
        python3-setuptools \
        python3-yaml \
        python3-zmq \
    && rm -rf /var/lib/apt/lists/*

COPY --from=compiling /usr/local/ /usr/local/

# make libraries known to dynamic linker
RUN ldconfig

LABEL org.opencontainers.image.source=https://github.com/akira25/gnuradio-docker-container
LABEL org.opencontainers.image.description="A minimal (headless) GNU Radio, packed into an docker container"
LABEL org.opencontainers.image.licenses=MIT
