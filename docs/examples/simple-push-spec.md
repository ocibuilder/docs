# Simple Push

An example of a simple **push** specification, which can be used as a `push.yaml` file or
configured within your `spec.yaml`.

```yaml
# You can either use this individual push.yaml file or 
# specify the push specifications in the global spec.yaml

# This will push your images to specified registries
push:
  - registry: abc.com
    image: my-awesome-build-ubuntu-xenial
    tag: v0.1.0
    login:
      inline:
        username: cool-username
        password: top-secret-password
  - registry: abc.com
    user: art
    image: my-awesome-build-2-ubuntu-xenial
    tag: v0.1.0
    login:
      env:
        username: $COOL_USERNAME
        password: $TOP_SECRET_PASSWORD
```