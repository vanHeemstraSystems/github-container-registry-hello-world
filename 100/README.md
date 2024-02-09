# 100 - Introduction

We follow the Continuous Integration (CI) - Continuous Delivery (CD) Workflow by means of Build Artifacts:

![CI-CD_Build_Artifacts](https://github.com/vanHeemstraSystems/github-container-registry-hello-world/assets/1499433/4e3a7c63-2169-468d-b7a1-e665113e2ceb)

The benefits of separating the CI part from the CD part using Build Artifacts are:

- You have **1:1 mapping between build and release**. This means that the image you build and the one you deploy are for sure the same.
- You can **directly reference the Build number**, or any other parameter that comes from the Build, because you CI and your CD pipelines are directly related.
- You have **full traceability**.
