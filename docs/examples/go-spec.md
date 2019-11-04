# Go Specification

An example of a well commented specification for a multi-stage build for a standard Go project.

```yaml
build:
  templates:
    - name: template-1
      cmd:
        - docker:
            inline:
              - ADD . /src
              - RUN cd /src && go build -o goapp
    - name: template-2
      cmd:
        - docker:
            inline:
              - WORKDIR /app
              - COPY --from=build-env /src/goapp /app/
              - ENTRYPOINT ./goapp
  steps:
    - metadata:
        name: docker.io/example-repo/go-service
        labels:
          type: build-1
          overlay: first-step
      stages:
        - metadata:
            name: build-env
          base:
            image: golang
            platform: alpine
          template: template-1
        - metadata:
            name: alpine-stage
          base:
            image: alpine
          template: template-2
      tag: v0.1.0
      purge: false
      context:
        localContext:
          contextPath: .

login:
  - registry: docker.io
    token: REPLACED_BY_PARAM
    creds:
      plain:
        username: REPLACED_BY_PARAM

push:
  - registry: docker.io
    image: example-repo/go-service
    tag: v0.1.0

params:
  - dest: login.0.token
    valueFromEnvVariable: DOCKERHUB_TOKEN
  - dest: login.0.creds.plain.username
    valueFromEnvVariable: DOCKERHUB_USER
```