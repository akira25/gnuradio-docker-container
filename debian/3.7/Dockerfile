FROM debian:10 AS compiler_with_deps_image

# activate archive package sources
RUN sed -i 's|deb\.|archive\.|g' /etc/apt/sources.list

# install build dependencies
RUN apt update && \
    apt install -y --no-install-recommends --no-install-suggests \
        wget git cmake build-essential nano \
        libboost-all-dev \
        libcodec2-dev \
        libcppunit-dev \
        libfftw3-dev \
        libgsl-dev \
        libgsm1-dev \
        libsdl-dev \
        libzmq3-dev \
        python-mako \
        python-numpy \
        python-six \
        python-zmq \
        python2-dev \
        swig \
    && rm -rf /var/lib/apt/lists/*


FROM compiler_with_deps_image AS compiling

WORKDIR /
RUN wget --no-check-certificate https://github.com/gnuradio/gnuradio/releases/download/v3.7.13.5/gnuradio-3.7.13.5.tar.gz
RUN tar xf gnuradio-3.7.13.5.tar.gz

WORKDIR /gnuradio-3.7.13.5/
RUN mkdir build
RUN cmake -B build -DCMAKE_C_FLAGS="-Wno-deprecated-declarations" \
      -DCMAKE_CXX_FLAGS="-Wno-deprecated-declarations" \
      -DENABLE_GR_AUDIO=OFF \
      -DCMAKE_BUILD_TYPE=RelWithDebInfo .
RUN cmake --build build --parallel $(($(nproc) + 2))
RUN cmake --strip --install build/


FROM debian:10 AS runtime_image

# activate archive package sources
RUN sed -i 's|deb\.|archive\.|g' /etc/apt/sources.list

# create virtual memory circular buffer (vmcircbuf) default config
RUN mkdir -p /root/.gnuradio/prefs
COPY <<EOF /root/.gnuradio/prefs/vmcircbuf_default_factory
gr::vmcircbuf_mmap_factory
EOF

# install runtime dependencies
RUN apt update && \
    apt install -y --no-install-recommends --no-install-suggests \
        libboost-all-dev \
        libcodec2-0.8.1 \
        libfftw3-3 \
        libfftw3-bin \
        libgsl23 \
        libgslcblas0 \
        libgsm1 \
        libsdl1.2debian \
        libzmq5 \
        python \
        python-mako \
        python-numpy \
        python-six \
        python-zmq \
    && rm -rf /var/lib/apt/lists/*

COPY --from=compiling /usr/local/ /usr/local/

# make libraries known to dynamic linker
RUN ldconfig

LABEL org.opencontainers.image.source=https://github.com/akira25/gnuradio-docker-container
LABEL org.opencontainers.image.description="A minimal (headless) GNU Radio, packed into an docker container"
LABEL org.opencontainers.image.licenses=MIT
