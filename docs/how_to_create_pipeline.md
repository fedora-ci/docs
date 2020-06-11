# How to create a new generic pipeline

This document describes how to create a new generic pipeline. Starting completely from scratch could be difficult, so this document describes how to take an existing generic pipeline and where to make changes that are specific to your test.

Note only standard Koji RPM builds and RPM build groups are supported at the moment. I.e. you're unfortunately out-of-luck if you'd like to test modules or other types of artifacts.

## Pipeline definition

The first step is to clone an existing pipeline and to rename it. Let's take the [rpmdeplint pipeline repository](https://github.com/fedora-ci/rpmdeplint-pipeline) for example:

```shell
git clone https://github.com/fedora-ci/rpmdeplint-pipeline.git mytest-pipeline
cd mytest-pipeline
rm -Rf .git/
```

There is a Jenkinsfile in the repository and the first thing we should do is to modify the pipeline metadata:

```groovy
def pipelineMetadata = [
    pipelineName: 'rpmdeplint',
    pipelineDescription: 'Finding errors in RPM packages in the context of their dependency graph',
    testCategory: 'functional',
    testType: 'rpmdeplint',
    maintainer: 'Fedora CI',
    docs: 'https://github.com/fedora-ci/rpmdeplint-pipeline',
    contact: [
        irc: '#fedora-ci',
        email: 'ci@lists.fedoraproject.org'
    ],
]
```

The pipeline uses the metadata internally, for example for publishing test results which contain information about the pipeline.

All fields in the list should be fairly self-explanatory, with exception of the "testCategory" and "testType" fields. You can find more information about these fields in the [Fedora CI messaging specification](https://pagure.io/fedora-ci/messages).

There are 2 stages in the pipeline. The first stage is called "Prepare" and as the name suggests, it just checks the input and performs some initialization steps. Usually you don't need to touch this stage.

The second stage is called "Test" and this is the place where you run your tests. If your test can run in Testing Farm (TODO: link once it is available) with help of the FMF definition (more on this later), then you can just tweak the parameters required by your test in this section and keep the rest as is.

If your test has some special requirements, then you can rewrite this stage of the pipeline completely. There are, however, some simple rules that you need to follow in order for the rest of the pipeline to work.
These rules are:

* if you run the tests and the final result is a failure, then you need to set the status to "UNSTABLE":

```groovy
currentBuild.result = 'UNSTABLE'
```

* if you couldn't run the tests or you couldn't run them properly (because there was some infrastructure failure for example), then you need to set the status to "FAILURE"

```groovy
currentBuild.result = 'FAILURE'
```

Setting the build status here is important as it determines which post-test sections will be executed next.

There are several post-test sections:

* always
* success
* failure
* unstable

These sections are responsible for handling various post-test tasks like publishing test results, archiving artifacts (if any), and so on. Generally you don't need to touch any of them.

### FMF Definition

It's important to describe your test via [FMF format](https://fmf.readthedocs.io/en/latest/) so the Testing Farm can later run it.
For simple cases (like generic tests), the format should be fairly easy to understand. Take a look at the existing [FMF file for the rpmdeplint](https://github.com/fedora-ci/rpmdeplint-pipeline/blob/master/rpmdeplint.fmf) and try to modify for your new generic pipeline.

### Pipeline Input

You might have noticed that the pipeline has a very simple input interface. It only takes two parameters:

* ARTIFACT_ID
* ADDITIONAL_ARTIFACT_IDS

The first parameter identifies an artifact that should be tested and the optional second parameter can be used if the testing environment should contain some additional artifacts.

For Koji builds, the artifact id has the following format:
`koji-build:<task-id>`

So for example `koji-build:42376994` is a valid input for the pipeline.

It's better not to add more parameters if not absolutely needed as more parameters make the pipeline harder to use.

## Triggers

Pipelines can be automatically triggered when certain events happen. For example, rpmdeplint pipeline triggers on Rawhide Koji builds.

To create a trigger for your new pipeline, the first step is to clone an existing pipeline and to rename it:

```shell
git clone https://github.com/fedora-ci/rpmdeplint-trigger.git mytest-trigger
cd mytest-trigger
rm -Rf .git/
```

Remember that the Jenkinsfile in the master branch is a trigger for Fedora Rawhide builds.
If you also just want to trigger on Rawhide builds, all you need to do is to rename the rpmdeplint reference in the Jenkinsfile:

```shell
sed -i 's|rpmdeplint|mytest|g' Jenkinsfile
```

And that's it.

You can of course trigger on different events, but then you will need to tweak the Jenkinsfile more.

## Container Image for the Generic Test

There is also a third rpmdeplint-related repository — [rpmdeplint-image](https://github.com/fedora-ci/rpmdeplint-image). A container image with the generic test can be built from this repository. It is then referenced from the FMF definition file and used by the Testing Farm.

There is nothing special about this repository. It's just the containerized version of the rpmdeplint test.
