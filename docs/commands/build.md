## Build

Builds an oci compliant image using either docker or buildah

#### Synopsis

This command runs an image build with the specification defined in your projects ocibuilder.yaml file. It can run a build in both docker and buildah varieties.

#### Command

```
ocictl build [flags] [...]
```

#### Options

```
-n, --name          Specify the name of your build or defined in ocibuilder.yaml
-p, --path          Path to your ocibuilder.yaml or build.yaml. By default will look in the current working directory
-b, --builder       Choose either docker and buildah as the targetted image builder. By default the builder is docker.
-d, --debug         Turn on debug logging
-o, --overlay       Path to your overlay.yaml file
```

#### Example

```
ocictl build --path ./ocibuilder.yaml --overlay ./overlay.yaml
```

or you can use shorthand options like this:

```
ocictl build -p ./ocibuilder.yaml -o ./overlay.yaml
```
