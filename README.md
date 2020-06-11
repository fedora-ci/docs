# Fedora CI pipelines for generic tests

## Pipeline Overview

Each generic Fedora CI pipeline consists of 2 or 3 repositories. The purpose of the repositories is as follows:

* repository containing pipeline and FMF definitions
  * example: [fedora-ci/rpmdeplint-pipeline](https://github.com/fedora-ci/rpmdeplint-pipeline)
* repository containing triggers for the pipeline
  * example: [fedora-ci/rpmdeplint-trigger](https://github.com/fedora-ci/rpmdeplint-trigger)
* (optional) repository containing instructions on how to containerize the test(s)
  * only needed if the generic test can run in a container
  * example: [fedora-ci/rpmdeplint-image](https://github.com/fedora-ci/rpmdeplint-image)

Branches in pipeline definition and trigger repositories follow the same naming conventions as branches in dist-git — i.e. "master" branch defines the pipeline for Fedora Rawhide, "f32" branch is for Fedora 32 and so on.
The container image repository is an exception and "master" branch is usually all that is needed.

## How to create a new generic pipeline

The process of creating a new generic pipeline is described in a separate document. You can find it [here](./docs/how_to_create_pipeline.md).