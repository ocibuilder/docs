# Simple Login

An example of a simple **login** specification, which can be used as a `login.yaml` file or
configured within your `ocibuilder.yaml`.

```yaml
# You can either use this individual login.yaml file or 
# specify the login specifications in the global ocibuilder.yaml

# login into different docker registries
login:
    - registry: test-registry
      creds:
        plain:
          username: cool-username
          password: top-secret-password
    - registry: url-of-the-registry
      creds:
        env:
          username: $COOL_USERNAME
          password: $TOP_SECRET_PASSWORD
```
