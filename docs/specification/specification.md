# Configuring Ocibuilder

This document is a reference for ocibuilder v0.1.0 specification keys used in `ocibuilder.yaml`.

You can find a complete `ocibuilder.yaml` example [here](./examples/complete-spec).

---

## Table of Contents
* [`build`](#build)
    * [`templates`](#templates)
        * [`name`](#name)
        * [`cmd`](#cmd)
            * [`ansible`](#ansible)
                * [`galaxy`](#galaxy)
                * [`local`](#local)
            * [`docker`](#docker)
    * [`steps`](#steps)
        * [`context`](#context)
            * [`localContext`](#localcontext)
            * [`gitContext`](#gitcontext)
        * [`stages`](#stages)
            * [`cmd`](#cmd)
            * [`metadata`](#metadata)
            * [`base`](#base)
        * [`metadata`](#metadata)
* [`login`](#login)
    * [`creds`](#creds)
        * [`plain`](#plain)
        * [`env`](#env)
* [`push`](#push)
* [`params`](#params)
* [`daemon`](#daemon)
* [`metadata`](#metadata)
    * [`storeConfig`](#storeconfig)
    * [`signKey`](#signkey)
        * [`grafeas`](#grafeas)
            * [`notes`](#notes)
    * [`data`](#data)
    
---

### `build`

A build comprises of reusable build templates and a number of build steps

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| templates | *(Array)* v1alpha1.BuildTemplate | Templates are set of build templates that describe steps needed to build a Dockerfile | No |
| steps | *(Array)* v1alpha1.BuildStep | Individual build definitions to run | Yes |


#### `templates`

Templates are reusable build configurations that can be used accross a number of different build steps and are referred to by the name field.
Multiple build commands can be entered which will be executed by ocibuilder.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| name | string | Name of the template | Yes |
| cmd | *(Array)* v1alpha1.BuildTemplateStep | Steps are instructions within a template to build a Dockerfile | Yes |


**Example**
```yaml
  templates:
    - name: go-build-template
      cmd:
        - docker:
            inline:
              - ADD . /src
              - RUN cd /src && go build -o goapp
```

The above example shows a very simple build template, using docker commands which have been passed inline.

#### `name`

Name is the name of the template. This can be referenced across multiple build steps and build stages allowing you to not have to rewrite 
standard build commands across different builds. 

#### `cmd`

Cmd allows you to specify commands that you want to run in your build. This can be standard docker commands, which can be passed 
inline or through a file. Alternatively, you are able to specify an ansible step which you can use to point to a local ansible playbook
or the name and requirements to pull from Ansible galaxy.


| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| ansible | v1alpha1.AnsibleStep | Ansible represents a ansible step within build template steps | No |
| docker | v1alpha1.DockerStep | Docker represents a docker step within build template steps | No |

**Example**
```yaml
  cmd:
    - docker:
        inline:
          - ADD . /src
          - RUN cd /src && go build -o goapp
    - docker:
        path: ./docker-commands.txt
    - ansible:
        galaxy:
          name: my-ansible-role
          requirements: ./requirements.yaml
        local:
          playbook: ./playbook.yaml
          
```

The above example shows all the possible flavours for inputting build commands into ocibuilder.

##### `ansible`

Ansible is used for entering references to local ansible playbooks or ansible roles in Ansible Galaxy. Each ansible step represents an
ansible install as part of the build.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| galaxy | v1alpha1.AnsibleGalaxy | Galaxy contains information to install a ansible role through ansible-galaxy | No |
| local | v1alpha1.AnsibleLocal | Local contains information to install a ansible role through local playbook | No |

##### `galaxy`

AnsibleGalaxy contains ansible role information to be installed at the build

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| name | string | Name of the galaxy role | Yes |
| requirements | string | Requirements refer to the requirements.yaml file | No |

##### `local`

AnsibleLocal contains information to install a ansible role through local playbook

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| playbook | string | Playbook refers to playbook.yaml file | Yes |


##### `docker`

Docker is used for entering docker commands that you want to execute in your build. You have the option of passing commands inline
with each docker command being a string array element.

Alternatively, you can pass in a filepath to a text file which contains your docker commands.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| inline | *(Array)* string | Inline Dockerfile commands | No |
| path | string | Path to a file that contains Dockerfile commands | No |
| url | string | Url to a remote command file | No |
| auth | v1alpha1.RemoteCreds | Basic auth for accessing a remote file  | No |

##### `auth`

You can specify remote basic auth credentials for accessing remote template files and overlays

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| username | string | Username refers to an env var that holds the username | Yes |
| password | string | Password refers to an env var that holds the password | No |

#### `steps`

Build *steps* are used to configure multiple unique builds with a single `spec.yml`. 

This can be particularly useful when trying to build multiple modules or 
projects in a single repository, allowing you to reuse the `templates` that you have defined in the specification.

Each step is run consecutively by ocibuilder with concurrent build steps running in a future
version of ocibuilder - progress can be tracked [here](https://github.com/ocibuilder/ocibuilder/issues/7).


| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| purge | boolean | Purge after built. Defaults is false. | No |
| context | v1alpha1.context | Specify an image build context for the step | No |
| stages | (*Array*) v1alpha1.Stage | Stages of the build | Yes |
| tag | string | The tag of the built image | No |
| metadata | v1alpha1.ImageMetadata | Image metadata, name, labels, annotations and source | No |

The purge flag allows you to purge your built images after they've been built. Ensures cleanup and is useful in build pipelines to
prevent the constant persisting of images which will not be used.

**Example**

```yaml
  # Steps are all the individual build steps you want to execute
  steps:
    # Metadata is where you define your final image build name as well as any labels
    - metadata:
        name: my-docker-registry:4555/art/go-service
      stages:
        - metadata:
            name: build-env
          base:
            image: golang
            platform: alpine
          template: go-build-template
      tag: v0.1.0
      purge: false
      context:
        localContext:
          contextPath: ./go-app
```

##### `context`

context enables you to specify a build context for each individual build step. For example, if you have
multiple directories and each directory is a separate build step, you are able to specify a particular
directory as the build context that step.

Additionally, an upcoming version of ocibuilder will have support to set external build contexts of S3 buckets and Git paths.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| localContext | context.LocalContext| Local context contains local context information for a build | No |

##### `localContext`

LocalContext holds local context information for an image build

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| contextPath | string | The path to your build context. | Yes |

##### `gitContext`

gitContext is the build context from git for an image build

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| url | string | The path to your build context | Yes |
| username | Credentials | Username for authentication | No |
| password | Credentials | Password for authentication | No |
| sshKeyPath | string | The path to your ssh key | No |
| branch | string | Branch to pull your context resource from | No |
| tag | string | Tag to pull your context resource from | No |
| ref | string | Ref to use to pull trigger resource. Will result in a shallow clone and fetch | No |
| remote | GitRemoteConfig | Remote to manage set of tracked repositories. Defaults to "origin" | No |

##### `stages`

Ocibuilder supports [docker multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) to help 
drastically reduce final image sizes. You are able to define a multi-stage build in each build step.

In each stage you have the option to define build commands under the `cmd` field or pass in a previously specified build template.

A stage also takes in a base image.

Required to pass in either cmd or a build template.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| base | v1alpha1.Base | Refers to parent image for given build stage. | Yes |
| cmd  | (*Array*) v1alpha1.BuildTemplateStep | Cmd refers to a template defined in a stage without a template. | No |
| template | string | Template refers to one of the build templates. | No |
| metadata | v1alpha1.ImageMetadata | Image metadata, name, labels, annotations and source | No |

**Example**

```yaml
  stages:
    - metadata:
        name: go-binary
      base:
        image: golang
        platform: alpine
      cmd:
        - docker:
            inline:
              - ADD . /src
              - RUN cd /src && go build -o goapp
    - metadata:
        name: alpine-stage
      base:
        image: alpine
      cmd:
        - docker:
            inline:
              - WORKDIR /app
              - COPY --from=go-binary /src/goapp /app/
              - ENTRYPOINT ./goapp
```

In the above example, the first stage of the build uses the golang:alpine base image to build our go binary and is named ``go-binary``.

Our second build stage refers to just our binary built in the first stage with ``--from=go-binary`` and copies this into our
new container image and sets an entrypoint.

More details about multi-stage builds can be found [here](../features/multi-stage-builds.md) 

##### `base`

Base is where you define your base image.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| image | string | The name of the image | Yes |
| platform | string | The specified platform of the image | No |
| tag | string | The tag for the image | No |

**Example**
```yaml
  base:
    image: golang
    platform: alpine
    tag: latest
```

##### `metadata` [`image`]

This is where any image metadata is defined, including image name, any labels and annotations that you want to specify for your build.

The metadata type is used both in build steps and build stages. 

Within a **build step** name is used to name your final built image, but can be also used to refer to other **build stages** in a
multi-stage build.

Any labels specified in your build configuration will be attached to your built image.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| annotations | map | Annotations for your build config | No |
| labels | map | Labels for your build config and built image | No |
| name | string | Name for your build configuration | Yes |
| creator | string | Creator is the creator of the project | Yes |
| source | string | Source is the URI to the source code repository | Yes |

---

### `login`

Login is used to specify credentials necessary to login to an image registry. These credentials are required to push and
pull images from an image registry.

You can specify credentials through plain, with environment variables, through the use of a token or a combination of the above.

[Params](#params) can also be used in conjunction with login credentials to pass in environment variables as a token or plaintext
credentials.

>**NOTE** Using a token may also require you to pass in a username as a credential depending on your image registry.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| creds | v1alpha1.RegistryCreds | credentials required to log into the registry | Yes |
| registry | string | Registry refers to a OCI image registry | Yes |
| token | string | An image registry token | No |

**Example**
```yaml
login:
  - registry: my-docker-registry:4555
    token: ThisIsMeloGinToken
    creds:
      plain:
        username: art
```

#### `creds`

Holds the specific credentials to login to a given image registry

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| env | v1alpha1.EnvCreds | Env refers to the credentials stored in environment variables | No |
| plain | v1alpha1.PlainCreds | Plain refers to the credentials set inline | No |

##### `env`

Credentials pulled from enviroment variables

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| username | string | Username refers to an env var that holds the username | Yes |
| password | string | Password refers to an env var that holds the password | Yes |

**Example**
```yaml
login:
  env:
    username: REGISTRY_USER
    password: REGISTRY_PASS
```

##### `plain`

Plaintext credentials for image registry login

>**NOTE** It is not recommended to store your credentials in plaintext

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| username | string | Plaintext registry username | Yes |
| password | string | Plaintext registry password | Yes |

**Example**
```yaml
login:
  env:
    username: artsuser
    password: artsp4ss
```

---

### `push`

Push enables you to specify multiple registries to push your image to. Push expects the registry you want to push to
have a corresponding login specification. 

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| image | string | Image to push | Yes |
| registry | string | Registry is the name of the registry | Yes |
| tag | string | Tag version of the image (e.g: v0.1.1) | Yes |

**Example**
```yaml
push:
  - registry: my-docker-registry:4555
    image: art/go-service
    tag: v0.1.0
```

---

### `params`

Params is where you can define parameters, allowing you to replace any value in the specification
with a value or an environment variable.

You specify the destination of the value you want to replace in the `dest` key using dot notation
to access nested elements.

>**NOTE**: A specific array item is referred to by index  in the dest field. For example, if you want to access the first step
element you would have ``steps.0``

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| dest | string | Dest is the destination of the field to replace with the parameter | Yes |
| value | string | Value of the environment variable. | No |
| valueFromEnvVariable | string |  a variable which is to be replaced by an env var | No |

**Example**
```yaml
params:
  # Replaces the value in location build.steps.0.tag with 0.0.3
  - dest: build.steps.0.tag
    value: 0.0.3
  # Replaces the value in location build.steps.0.metadata.name with the environment variable $BUILD_DEV
  - dest: build.steps.0.metadata.name
    valueFromEnvVariable: BUILD_DEV
```

### `daemon`

Daemon is a boolean which allows you to specify whether you want to use the docker daemon, or buildah as an alternative.

A `true` value will execute all commands using docker, a `false` flag will execute all commands using buildah.

This value can be overriding by the `builder` command line flag which takes priority.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| daemon | boolean | Allows you to specify whether to use the docker daemon as a builder or buildah. Default is true. | No |

### `metadata`

Metadata allows you to define all the configurations for pushing build and image metadata into a metadata store like [Grafeas](https://github.com/grafeas/grafeas).

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| storeConfig | v1alpha1.StoreConfig | StoreConfig holds the configuration for connecting to a metadata store | No |
| signKey | *v1alpha1.SignKey | SignKey holds the signing key for signing image IDs for image attestation purposes | No |
| hostname | string | Hostname is the hostname of your metadata store | No |
| data | []v1alpha1.MetadataType | Data holds a list of all the metadata types that you would like to push to your metadata store | No |

**Example**
```yaml
metadata:
  hostname: "http://localhost:8080"
  signKey: 
    envPublicKey: OCI_PUB_KEY
    envPrivateKey: OCI_PRI_KEY
  storeConfig:
    grafeas:
      project: "image-build"
      notes:
        build: "projects/image-build/notes/oci-build"
        attestation: "projects/image-build/notes/oci-attest"
        image: "projects/image-build/notes/oci-image"
  data:
    - image
    - build
    - attestation
```

#### `storeConfig`

Store config holds store specific configurations to do with connection and required fields. Currently the only metadata store 
support by the ocibuilder is Grafeas.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| grafeas | *v1alpha1.Grafeas | Grafeas holds the config for the Grafeas metadata store | Yes |

##### `grafeas`

Grafeas holds any grafeas specific configurations. This allows you to specify a `project` to push metadata to
as well as define individual `notes` for each type of metadata that can be pushed.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| project | string | Project is the name of the project ID to store the occurrence | Yes |
| notes | v1alpha1.Notes | Notes holds the notes for each of the three occurrence types | Yes |

###### `notes`

Notes is where you can specify Grafeas note names for Build, Attestation and DerivedImage metadata.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| attestation | string | Attestation is the analysis note associated with a attestation occurrence, in the form of `projects/[PROVIDER_ID]/notes/[NOTE_ID]` | Yes |
| build | string | Build is the analysis note associated with a build occurrence, in the form of `projects/[PROVIDER_ID]/notes/[NOTE_ID]` | Yes |
| image | string | Image is the analysis note associated with a derived image occurrence, in the form of `projects/[PROVIDER_ID]/notes/[NOTE_ID]` | Yes |


#### `signKey`

SignKey holds your private and public signing keys to sign your image after it has been built which can be used later for image attestation.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| plainPrivateKey | string | PrivateKey is an ascii armored private key used to sign images for image attestation | No |
| plainPublicKey | string | PublicKey is the ascii armored public key for verification in image attestation | No |
| envPrivateKey | string | EnvPrivateKey is an env variable that holds an ascii armored private key used to sign images for image attestation | No |
| envPublicKey | string | EnvPublicKey is an env variable that holds an ascii armored public key used to sign images for image attestation | No |
| passphrase | string | Passphrase is the passphrase for decrypting the private key | No |

#### `data`

Data holds a list of the types of metadata that you want pushed to the Metadata store.

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| attestation | string | Attestation in the data array signifies you want to store attestation metadata | No |
| build | string | Build in the data array signifies you want to store build metadata | No |
| image | string | Image in the data array signifies you want to store image metadata | No |
