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

Download a binary release of ocictl from the [oficial releases page](https://github.com/ocibuilder/ocibuilder/releases) or pull the ocictl image from
our [Docker registry](https://cloud.docker.com/u/ocibuilder/repository/docker/ocibuilder/ocictl).

More details are available on the ocictl [installation guide](./INSTALL.md)

## First Steps

Once you have `ocictl` installed and available in the path, you can now put together your desired build specification.

The quickest way to get a sample spec within your project is to run:

```ocictl init```

This will generate a template specification for you which you can then fill in with your custom build specifications.

Our [specification docs](https://github.com/ocibuilder/docs/blob/master/spec/specification.md) outline in detail how to tailor your specification
to your specific build.