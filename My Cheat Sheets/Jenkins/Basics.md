### What is a Jenkins Pipeline?[](https://www.jenkins.io/doc/pipeline/tour/hello-world/#what-is-a-jenkins-pipeline)

Jenkins Pipeline (or simply "Pipeline") is a suite of plugins which supports implementing and integrating _continuous delivery pipelines_ into Jenkins.

A _continuous delivery pipeline_ is an automated expression of your process for getting software from version control right through to your users and customers.

Jenkins Pipeline provides an extensible set of tools for modeling simple-to-complex delivery pipelines "as code". The definition of a Jenkins Pipeline is typically written into a text file (called a `Jenkinsfile`) which in turn is checked into a project’s source control repository. [[1](https://www.jenkins.io/doc/pipeline/tour/hello-world/#_footnotedef_1 "View footnote.")]

# Running multiple steps [](https://www.jenkins.io/doc/pipeline/tour/running-multiple-steps/#running-multiple-steps)
Pipelines are made up of multiple steps that allow you to build, test and deploy applications. Jenkins Pipeline allows you to compose multiple steps in an easy way that can help you model any sort of automation process.

Think of a "step" like a single command which performs a single action. When a step succeeds it moves onto the next step. When a step fails to execute correctly the Pipeline will fail.

When all the steps in the Pipeline have successfully completed, the Pipeline is considered to have successfully executed.

### Linux, BSD, and Mac OS[](https://www.jenkins.io/doc/pipeline/tour/running-multiple-steps/#linux-bsd-and-mac-os)

On Linux, BSD, and Mac OS (Unix-like) systems, the `sh` step is used to execute a shell command in a Pipeline.

Jenkinsfile (Declarative Pipeline)

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'echo "Hello World"'
                sh '''
                    echo "Multiline shell steps works too"
                    ls -lah
                '''
            }
        }
    }
}
```

[Toggle Scripted Pipeline](https://www.jenkins.io/doc/pipeline/tour/running-multiple-steps/#) _(Advanced)_

### [](https://www.jenkins.io/doc/pipeline/tour/running-multiple-steps/#windows)Windows[](https://www.jenkins.io/doc/pipeline/tour/running-multiple-steps/#windows)

Windows-based systems should use the `bat` step for executing batch commands.

Jenkinsfile (Declarative Pipeline)

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                bat 'set'
            }
        }
    }
}
```