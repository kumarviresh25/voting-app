pipeline {
    agent any

    stages {
        stage('Verify Branch') {
            steps {
                sh 'echo "$GIT_BRANCH"'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker images -a'
                sh '''
                   cd azure-vote/
                   docker images -a
                   docker build -t jenkins-pipeline .
                   docker images -a
                   cd ..
                '''
            }
        }
        stage('Start test app') {
            steps {
                sh '''
                   docker-compose up -d
                   powershell ./scripts/test_container.ps1
                '''
            }
            post {
                success {
                    echo "App started successfully :)"
                }
                failure {
                    echo "App failed to start :("
                }
            }
        }
        stage('Run Tests') {
            steps {
                sh '''
                   pytest ./tests/test_sample.py
                '''
            }
        }
        stage('Stop test app') {
            steps {
                sh '''
                   docker-compose down
                '''
            }
        }
        stage('Container Scanning') {
            parallel {
                stage('Run Anchore') {
                    steps {
                        sh '''
                           echo "blackdentech/jenkins-course" > anchore_images
                        '''
                        // You can replace the above sh step with your actual shell commands.
                    }
                }
                stage('Run Trivy') {
                    steps {
                        sleep(time: 30, unit: 'SECONDS')
                        // Add your shell commands for Trivy scanning here.
                    }
                }
            }
        }
        stage('Deploy to QA') {
            environment {
                ENVIRONMENT = 'qa'
            }
            steps {
                echo "Deploying to ${ENVIRONMENT}"
                acsDeploy(
                    azureCredentialsId: "jenkins_demo",
                    configFilePaths: "**/*.yaml",
                    containerService: "${ENVIRONMENT}-demo-cluster | AKS",
                    resourceGroupName: "${ENVIRONMENT}-demo",
                    sshCredentialsId: ""
                )
            }
        }
        stage('Approve PROD Deploy') {
            when {
                branch 'master'
            }
            options {
                timeout(time: 1, unit: 'HOURS') 
            }
            steps {
                input message: "Deploy?"
            }
            post {
                success {
                    echo "Production Deploy Approved"
                }
                aborted {
                    echo "Production Deploy Denied"
                }
            }
        }
        stage('Deploy to PROD') {
            when {
                branch 'master'
            }
            environment {
                ENVIRONMENT = 'prod'
            }
            steps {
                echo "Deploying to ${ENVIRONMENT}"
                acsDeploy(
                    azureCredentialsId: "jenkins_demo",
                    configFilePaths: "**/*.yaml",
                    containerService: "${ENVIRONMENT}-demo-cluster | AKS",
                    resourceGroupName: "${ENVIRONMENT}-demo",
                    sshCredentialsId: ""
                )
            }
        }
    }
}

