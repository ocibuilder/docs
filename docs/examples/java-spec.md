# Java Specification

An example of a well commented specification for a multi-stage build for a standard Java war.

```yaml
build:
  templates:
    - name: template-1
      cmd:
        - docker:
            inline:
                # copy WAR into image
                - COPY spring-boot-app-0.0.1-SNAPSHOT.war /app.war 
                # run application with this command line 
                - CMD ["/usr/bin/java", "-jar", "-Dspring.profiles.active=default", "/app.war"]
  steps:
    - metadata:
        name: docker.io/example-repo/java-proj
      stages:
        - metadata:
            name: build-env
          base:
            image: openjdk
            tag: 8-jre-alpine
          template: template-1
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
    image: example-repo/java-proj
    tag: v0.1.0

params:
  - dest: login.0.token
    valueFromEnvVariable: DOCKERHUB_TOKEN
  - dest: login.0.creds.plain.username
    valueFromEnvVariable: DOCKERHUB_USER
```