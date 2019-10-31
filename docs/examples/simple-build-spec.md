# Simple Build

An example of a simple **build** specification, which can be used as a `build.yaml` file or
configured within your `spec.yaml`.

```yaml
# You can either use this individual build.yaml file or 
# specify the build specifications in the global spec.yaml

builds:
    templates:
      - name: template-1
        cmd:
          - docker:
              inline:
                - RUN pip install kubernetes
                - COPY app/ /bin/app
    steps:
      - metadata:
          name: name-of-the-build
          labels:
            type: build-1
        stages:
          - metadata:
              name: name-of-the-stage
            base:
              image: python
              tag: 3
            template: template-1
        tag: v0.1.0
        purge: true
        daemon: true
```
