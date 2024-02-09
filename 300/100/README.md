# 100 - Code for minimal example

## Content of ```Dockerfile```

```
ARG BUILD_FROM
FROM $BUILD_FROM

# Install requirements for add-on
RUN \
  apk add --no-cache \
    python3

# Python 3 HTTP Server serves the current working dir
# So let's set it to our add-on persistent data directory.
WORKDIR /data

# Copy data for add-on
COPY run.sh /
RUN chmod a+x /run.sh

CMD [ "/run.sh" ]
```

Dockerfile

## Content of ```config.yaml```

```
name: "Hello world"
description: "My first real add-on!"
version: "1.2.0"
slug: "hello_world"
init: false
options:
  beer: true
  wine: true
  liquor: false
  name: "world"
  year: 2017
schema:
  beer: bool
  wine: bool
  liquor: bool
  name: str
  year: int
arch:
  - aarch64
  - amd64
  - armhf
  - armv7
  - i386
startup: services
ports:
  8001/tcp: 8001
```

config.yaml

## Content of ```run```

```
#!/usr/bin/with-contenv bashio

echo "Hello world!"

python3 -m http.server 8001
```

run.sh

## Content of ```.gitlab-ci.yaml```

```
stages:
  - build
  - release

build-job:
  stage: build
  script:
    - echo "Compiling the code..."
    # get the job id and save it as environment statement
    - echo BUILD_JOB_ID=$CI_JOB_ID >> CI_JOB_ID.env
    - mkdir build
    - cat fileconc1.txt fileconc2.txt > build/file.txt
    - echo "Compile completed."
  artifacts:
    paths:
    - "build/file.txt"
    reports:
      # export the environment statement so we can access it in the release stage
      dotenv: CI_JOB_ID.env
release-job:
  stage: release
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  needs:
    - job: build-job
      artifacts: true
  script:
    - echo "release application..."
    - echo "Application successfully released."
  release:
    name: 'Release Executables $CI_COMMIT_SHORT_SHA'
    description: 'Created using the release-cli'
    # tag_name is a memandatory field and can not be an empty string
    tag_name: '$CI_COMMIT_SHORT_SHA'
    assets:
      links:
        - name: 'File from cat'
          # Use variables to build a URL to access the artifacts
          # ${CI_PROJECT_URL} is the repository URL
          # ${BUILD_JOB_ID} is from the previous job,
          # the build stage, that contains the artifact
          url: '${CI_PROJECT_URL}/-/jobs/${BUILD_JOB_ID}/artifacts/file/build/file.txt'
  only:
    - main # only execute the release if the pipeline runs for the main branch
```

.gitlab-ci.yaml
