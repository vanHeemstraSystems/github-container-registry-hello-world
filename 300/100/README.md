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

## Content of ```.gitlab-ci.yml```

**NOTE**: GitLab will automatically recognize ```.gitlab-ci.yml``` as a pipeline file, whereas it won't recognize ```.gitlab-ci.yaml``` as a pipeline. Mind the ```.yml``` spelling therefor!

```
# This file is a template, and might need editing before it works on your project.
# This is a sample GitLab CI/CD configuration file that should run without any modifications.
# It demonstrates a basic 3 stage CI/CD pipeline. Instead of real tests or scripts,
# it uses echo commands to simulate the pipeline execution.
#
# A pipeline is composed of independent jobs that run scripts, grouped into stages.
# Stages run in sequential order, but jobs within stages run in parallel.
#
# For more information, see: https://docs.gitlab.com/ee/ci/yaml/index.html#stages
#
# You can copy and paste this template into a new `.gitlab-ci.yml` file.
# You should not add this template to an existing `.gitlab-ci.yml` file by using the `include:` keyword.
#
# To contribute improvements to CI/CD templates, please follow the Development guide at:
# https://docs.gitlab.com/ee/development/cicd/templates.html
# This specific template is located at:
# https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Getting-Started.gitlab-ci.yml

stages:          # List of stages for jobs, and their order of execution
  - build
  - test
  - release
  - deploy

build-job:       # This job runs in the build stage, which runs first.
  stage: build
  script:
    - echo "Compiling the code..."
    # get the job id and save it as environment statement
    - echo BUILD_JOB_ID=$CI_JOB_ID >> CI_JOB_ID.env
    - mkdir build
    - cat LICENSE > build/LICENSE
    - cat README.md > build/README.md
    - cat Dockerfile > build/Dockerfile
    - cat config.yaml > build/config.yaml
    - cat run.sh > build/run.sh
    - cd build
    - tar -cf hello_world.tar .
    - cd ../    
    - echo "Compile complete."
  artifacts:
    paths:
    - "build/hello_world.tar"
    reports:
      # export the environment statement so we can access it in the release stage
      dotenv: CI_JOB_ID.env    

unit-test-job:   # This job runs in the test stage.
  stage: test    # It only starts when the job in the build stage completes successfully.
  script:
    - echo "Running unit tests... This will take about 60 seconds."
    - sleep 60
    - echo "Code coverage is 90%"

lint-test-job:   # This job also runs in the test stage.
  stage: test    # It can run at the same time as unit-test-job (in parallel).
  script:
    - echo "Linting code... This will take about 10 seconds."
    - sleep 10
    - echo "No lint issues found."

release-job:      # This job runs in the release stage.
  stage: release  # It only runs when *both* jobs in the test stage complete successfully.
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
    # tag_name is a mandatory field and cannot be an empty string
    tag_name: '$CI_COMMIT_SHORT_SHA'
    assets:
      links:
        - name: 'hello_world'
          # Use variables to build a URL to access the artifacts
          # ${CI_PROJECT_URL} is the repository URL
          # ${BUILD_JOB_ID} is from the previous job,
          # the build stage, that contains the artifact
          url: '${CI_PROJECT_URL}/-/jobs/${BUILD_JOB_ID}/artifacts/file/build/hello_world.tar'
  only:
    - main # only execute the release if the pipeline runs for the main branch

deploy-job:      # This job runs in the deploy stage.
  stage: deploy  # It only runs when *both* jobs in the test stage complete successfully.
  environment: production
  script:
    - echo "Deploying application..."
    - echo "Application successfully deployed."
```

.gitlab-ci.yml

The ```.gitlab-ci.yml``` file defines five stages, "build", "test", "release", and "deploy". 

The “**build**” stage compiles the code by copying the content of the files to a directory named “build” and archives them as "hello_world.tar".

It also creates an environment statement ```BUILD_JOB_ID=$CI_JOB_ID``` using the job ID to use it later in the release stage. 

In the artifact section, it exports the environment statement so we can access it in the release stage.

## Crafting the URL

One tricky part of using GitLab CI/CD Releases with build is generating the URL to access the artifact in the release stage. In the example given, the URL is generated using a combination of the GitLab project URL (CI_PROJECT_URL), the job ID of the build stage (BUILD_JOB_ID), and the artifact path (/artifacts/file/build/hello_world.tar).

To obtain the BUILD_JOB_ID, we used an environment statement in the build stage: BUILD_JOB_ID=$CI_JOB_ID. This statement saves the job ID of the build stage in an environment variable named BUILD_JOB_ID. We then exported this environment variable using the artifact section dotenv: CI_JOB_ID.env so that we could access it in the release stage.

Using this environment variable, we can generate the URL to access the artifact in the release stage by combining it with the GitLab project URL and the artifact path. In the example given, the artifact path is /artifacts/file/build/hello_world.tar, which specifies the location of the archive in the artifact archive.

The resulting URL would look something like this: https://gitlab.com/myusername/myproject/-/jobs/123/artifacts/file/build/hello-world.tar, where myusername is the username of the GitLab user, myproject is the name of the GitLab project, 123 is the job ID of the build stage, and /artifacts/file/build/hello_world.tar is the path to the artifact archive.

By using this URL, users can easily download the artifact that was created in the build stage. This is useful for distributing software releases or other types of files to users.

In conclusion, generating the URL to access artifacts in the release stage can be a bit tricky, but by using environment variables like BUILD_JOB_ID and combining them with the GitLab project URL and artifact path, we can easily create a URL that users can use to download our artifacts.

The “**release**” stage releases the application using the GitLab release-cli image. It requires the “build” stage to be executed successfully by using needs. 

The script in the release stage echoes the release application and successfully released the application. In the release section, it creates a name and description for the release and specifies the tag name. It also specifies the URL to access the artifacts in the assets section.
