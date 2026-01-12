# GNU Radio in a minimal-sized Container

This repository shows a concept for packaging GNU-Radio flowgraphs into Linux docker containers. Compared to other approaches, we focus on small container size and enhanced modularity.

## Supported tags

Tags comprise the GNU Radio version and the respective base image. For example, `3.10-alpine` denotes a minimal GNU Radio 3.10 on an alpine base.

The `latest` tag will always point to the newest GNU Radio version, built on an alpine base image.

## TL;DR: Test it out

This demonstration assumes that you have already installed a recent GNU Radio on your computer to handle the hardware drivers, sound, etc. You will need this to execute the host flowgraph.

```sh
# checkout github repo for demo
$ git clone https://github.com/Akira25/gnuradio-docker-container.git

# pull the image
$ docker pull akira25/gnuradio-docker-container:test

# start the container directly in the host network
$ docker run --rm --network=host akira25/gnuradio-docker-container:test

# start the host flowgraph for audio control
$ ./receiver.py
```

You can find further information on this demo in the section _Demonstration_.

## Architectural goals

We want to achieve running even old flowgraphs seamlessly, as their dependencies get included in the container. This might come in very handy in the long term.

The flowgraphs in such a container should contain ZMQ-Blocks (or plain TCP/UDP, if you wish) for their input and output. I.e. like this:

![Example flowgraph with ZMQ-Blocks for their input and output](flowgraph.png)

We consider this useful in different scenarios:

- Decouple your GNU Radio application from the underlying OS and its GNU Radio version
- Combine blocks from different GNU Radio versions
- Spawn different flowgraphs in your transmission pipeline quickly, i.e. for a GNU Radio-based transceiver working in different modes

## Limitations

We expect the containers to be run in the host network. That simplifies communications, as you can talk to the flowgraphs over plain sockets.

To circumvent complicated redirections, e.g., with audio or USB devices, we expect additional flowgraphs using ZMQ-Blocks, which control audio, hardware, and so on, to be run directly on the Host system without a Docker container.

_Improving Performance: Using TCP as base for the transport mechanism holds some unessesary overhead, when you stay on the same host machine. TCPs Slow-Start machnism will ZMQ force to drop some samples at startup. I published a refined approach using unix-domain-sockets at [akira25/gnuradio-docker-container-unix-sockets](https://github.com/akira25/gnuradio-docker-container-unix-sockets)._

## Demonstration

To build a demonstration on your computer, follow this section's steps. We used `podman` as the container runtime. Still, you should be able to run these commands perfectly with docker by just replacing `podman` with the `docker` command:

```sh
# Build the container from Dockerfile
$ podman build .
# Start the docker container in the host network
$ podman run --network=host 59f3c95578b3  # substitue this with the ID of your image
# Start the host-part of the flowgraph handling the audio:
$ ./receiver.py
```

You should also start the `receiver` flowgraph from your local GNU Radio companion. By running this setup, you can hear a binaural beat at 440 Hz. The signal is generated within the flowgraph and sent via the ZMQ sockets to the host part.

The Dockerfile at `/alpine/3.10/Dockerfile` generates the docker image containing the GNU Radio runtime. Depending on your build host, this takes roughly 15 to 30 minutes.

## Further Notes

This repository is a proof of concept that can be improved further. At the time of writing, an Alpine-Linux image with the stripped down GNU Radio takes around 260MB only! We expect this to be enhanced when restraining the build to those modules that are actually used in the application flowgraph only. For example, if a flowgraph does not deal with blocks from `gr-digital`, this module does not need to be included.
