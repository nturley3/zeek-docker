# Zeek Docker
Zeek Docker builds for learning Zeek, script development, testing or running Zeek at home!

# Getting Started
This repository contains necessary files for building and running Zeek on Docker. Multiple builds are provided in this repository for different situations. There are some Docker best practices and optimizations that have ***not*** been added since this repository was intended for learning, development, testing and utlimately used by students. 

There are two versions of build files contained in this repository:

* An Ubuntu build version that is used for Zeek script development, code testing, PCAP testing etc. This is not a multi-stage build and produces a rather large image (around 1GB). It contains some helpful tools for doing Zeek script development form within the container. This will compile the latest LTS version of Zeek. 
* An Alpine build that is multi-stage and can be used to run Zeek in your environment on live traffic. For example, this build is great for running Zeek on a Raspberry Pi at home. This will compile the latest LTS version of Zeek and the image size is around 40MB. 

NOTE: At this time, a trusted build is not available from a public Docker Registry. You should use this repository to build local images. 

This repository is primarily for LTS versions of Zeek 4.0+. 

# Building Zeek Docker Images
## Ubuntu Build

1. Change your directory to `<repo_path>/4.0/ubuntu`. You will find a `docker-compose.yml` file. You can build the image manually, but docker-compose is provided for convenience. 
2. Run: `docker-compose build`. The build will take some time. 
3. Once the build finishes, you should have a new image called `zeek-docker-ubuntu:4`. 

At this point, there are two ways to run the container: `docker-compose` or `docker run`. 

**docker-compose**

If you would like to use docker-compose to run the image, or run Zeek on live interface traffic, run the following:

```
docker-compose up -d
```

**docker run**

To run the container manually, run:

```
docker run --rm -ti zeek-docker-ubuntu:4 /bin/zsh
```

When you run the container in this fashion, some helpful information will be reported:
```
Zeek version: 
zeek version 4.0.3

zkg config: 
[sources]
zeek = https://github.com/zeek/packages

[paths]
state_dir = /opt/zeek/var/lib/zkg
script_dir = /opt/zeek/share/zeek/site
plugin_dir = /opt/zeek/lib/zeek/plugins
zeek_dist = /app/zeek-source/zeek-4.0.3


/opt/zeek/logs #                 
```

## Alpine Build

1. Change your directory to `<repo_path>/4.0/alpine`. You will find a `docker-compose.yml` file. You can build the image manually, but docker-compose is provided for convenience. 
2. Run: `docker-compose build`. The build will take some time. 
3. Once the build finishes, you should have a new image called `zeek-docker-alpine:4`. 

Since the Alpine image is intended to run on live traffic (such as a Raspberry Pi), it has been stipped down and contains no user shells or utilities. You should use `docker-compose`: 

**docker-compose**

```
docker-compose up -d
```

You can verify the container has started correctly and is listening on your host interface. 
```
docker-compose logs
```

You should see Zeek listening on the selected interface. For example:
```
zeek-docker-alpine | listening on eth0
```

NOTE: The `docker-compose.yml` file defaults to the `eth0` interface. If your server or workstation doesn't have `eth0` defined, running the container will fail with an error like the following:

```
zeek-docker-alpine | fatal error: problem with interface af_packet::eth0 (No such device)
```

This usually means the `eth0` interface is named something else on your host (for example, `enx381428befbf2`). Use `ifconfig` or `ip addr` to see the name of the interface. Since this container uses "host" mode networking configuration, you must have the correct name on your host for the interface for which you want Zeek to listen. 



