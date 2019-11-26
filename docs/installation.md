# Installation

Binary downloads of the `ocictl` are available on the [Releases page](https://github.com/ocibuilder/ocibuilder/releases).


You can use the `install.sh` script to install the latest version of `ocictl`:

```bash
curl https://raw.githubusercontent.com/ocibuilder/ocibuilder/master/install.sh | sh
```

This requires `GOPATH` to be set, with bin added to your `PATH`.

The latest images with Buildah and Docker pre-installed alongside the ocictl is available on our 
[Dockerhub repository](https://cloud.docker.com/u/ocibuilder/repository/docker/ocibuilder/ocictl).

To pull the latest image run the following command:

```bash
docker pull ocibuilder/ocictl:latest
```
