#
# This builds a custom gnuradio container on alpine basis in two stages:
#
# First compile a stripped down gnuradio version, without GRC, intended
# to be run headless
#
# Then, build a container from that.
#

FROM alpine:3.21 AS build
WORKDIR /gnuradio
RUN apk add alpine-sdk doas

# Create a abuild group and user https://stackoverflow.com/a/49955098
RUN adduser --disabled-password abuild -G abuild \
        && addgroup abuild wheel

# Configure doas to be used by wheel users
COPY <<EOF /etc/doas.d/wheel_becomes_root.conf
permit nopass :wheel
EOF

RUN chown -R abuild:abuild /gnuradio
COPY . /gnuradio

USER abuild
RUN abuild-keygen -a -i -n
RUN abuild -r

# step image with installing custom gnuradio version
FROM alpine:3.21 AS alpine_gr

RUN apk add --no-cache zeromq py3-pyzmq

COPY --from=build /home/abuild/packages/*/gnuradio-*.apk /gnuradio_apks/
RUN apk add --allow-untrusted --no-cache /gnuradio_apks/gnuradio-*.apk

LABEL org.opencontainers.image.source=https://github.com/Akira25/gnuradio-docker-container
LABEL org.opencontainers.image.description="A minimal (headless) GNU Radio, packed into an alpine docker container"
LABEL org.opencontainers.image.licenses=MIT
