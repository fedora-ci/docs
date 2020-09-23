# Pipeline Linter

[Pipeline linter](https://www.jenkins.io/doc/book/pipeline/development/#linter)

Jenkins can validate, or "lint", a Declarative Pipeline from the command line before actually running it.
This can be done using a Jenkins CLI command or by making an HTTP POST request with appropriate parameters.

For example:
```
MY_JENKINSFILE=validated_jenkinsfile
MY_JENKINS_URL=https://my-jenkins-project.org

curl -H "Authorization: OAuth " -X POST -F "jenkinsfile=<$MY_JENKINSFILE" $MY_JENKINS_URL/pipeline-model-converter/validate
```
