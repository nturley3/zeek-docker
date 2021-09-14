# Zeek Docker
Zeek Docker builds for learning Zeek, script development, testing or running Zeek at home!

# Getting Started
This repository contains necessary files for building and running Zeek on Docker. Multiple builds are provided in this repository for different situations. There are some Docker best practices and optimizations that have ***not*** been added since this repository was intended for learning, development, testing and utlimately used by students. 

There are two versions of build files contained in this repository:

* An Ubuntu build version that is used for Zeek script development, code testing, PCAP testing etc. This is not a multi-stage build and produces a rather large image (around 1GB). It contains some helpful tools for doing Zeek script development form within the container. This will compile the latest LTS version of Zeek. 
* An Alpine build that is multi-stage and can be used to run Zeek in your environment on live traffic. For example, this build is great for running Zeek on a Raspberry Pi at home. This will compile the latest LTS version of Zeek. 

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

TBD