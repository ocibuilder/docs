# Quickstart Guide

This guide covers how you can quickly get started using OCI builder.

## Prerequisites

The prerequisites for running ocibuilder are as standard

If running `ocictl` as standalone:
1. Have a docker installed and the docker daemon running
*and/or*
2. Have buildah installed and available in `$PATH`

Alternatively, if you're using the [ocictl image](https://cloud.docker.com/u/ocibuilder/repository/docker/ocibuilder/ocictl) you will
have both Docker and Buildah installed by default but with the following consideration:

* You must run the image in an environment which supports Docker in Docker is using Docker as a builder.

#### Install Docker

Docker can be easily installed by following the instructions for the installation of [Docker Desktop](https://www.docker.com/products/docker-desktop)

#### Install Buildah

Buildah can be installed for a number of different linux distributions and instructions are available [here](https://github.com/containers/buildah/blob/master/install.md)

## Install ocictl

You can use the `install.sh` script to install the latest version of `ocictl`:

```bash
curl https://raw.githubusercontent.com/ocibuilder/ocibuilder/master/install.sh | sh
```

This requires `GOPATH` to be set, with bin added to your `PATH`.

Alternatively, you can pull the latest image with Buildah and Docker pre-installed alongside the ocictl is available on our 
[Dockerhub repository](https://cloud.docker.com/u/ocibuilder/repository/docker/ocibuilder/ocictl).

To pull the latest image run the following command:

```bash
docker pull ocibuilder/ocictl:latest
```

More details are available on the ocictl [installation guide](./installation.md)

## Generating Template Spec

Once you have `ocictl` installed and available in the path, you can now put together your desired build specification.

The quickest way to get a sample spec within your project is to run:

```bash
ocictl init
```

This will generate a template specification for you which you can then fill in with your custom build specifications.

Our [specification docs](https://github.com/ocibuilder/docs/blob/master/spec/specification.md) outline in detail how to tailor your specification
to your specific build.

## Running a Build

Once you have configured your build specification to your project you can conduct a build by running:

```bash
ocictl build
```

This will conduct a docker build be default with your given specification. By default your build context path will be treated as the current
directory.

If you want to run the build using `buildah` as the build tool, you can:

1. In your specification, set the `daemon` field to `false`
2. Run the following command instead:

```bash
ocictl build --builder buildah
```

## Pushing your built image

Once your image has been built you can very easily push it with the following command

```bash
ocictl build
```


