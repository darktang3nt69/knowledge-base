### 1. On Linux,  use sh command to execute shell commands:
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

### 2. Timeouts and Retries:
```groovy
pipeline {
    agent any
    options {
        retry(5) #  Retry the entire pipeline 5 times.
    }
    stages {
        stage('Deploy') {
            steps {
                retry(3) {                           # The "Deploy" stage retries the `flakey-deploy.sh` script 3 times
                    sh './flakey-deploy.sh'
                }

                timeout(time: 3, unit: 'MINUTES') {  # waits for up to 3 minutes for the `health-check.sh` script to execute.
                    sh './health-check.sh'           # If the health check script does not complete in 3 minutes, 
                                                     # the Pipeline will be marked as having failed in the "Deploy" stage.
                }
            }
        }
        stage('Retry deployment 5 times, but never want to spend more than 3 minutes in total before failing the stage'){
            steps{
                timeout(time: 3, unit: 'MINUTES'){
                    retry(5){
                        sh './deploy.sh'
                    }
                }
            }
        }
    }
}
```

### 3. Post steps: 
- When the Pipeline has finished executing, you may need to run clean-up steps or perform some actions based on the outcome of the Pipeline. These actions can be performed in the `post` section.
```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'echo "Fail!"; exit 1'
            }
        }
    }
    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}
```

### 4. Using ENV vars: 
- Environment variables can be set globally, like the example below, or per stage. As you might expect, setting environment variables per stage means they will only apply to the stage in which they’re defined.
```groovy
pipeline {
    agent {
        label '!windows'
    }

    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }

    stages {
        stage('Build') {
            steps {
                echo "Database engine is ${DB_ENGINE}"
                echo "DISABLE_AUTH is ${DISABLE_AUTH}"
                sh 'printenv'
            }
        }
    }
}
```