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
        // Add more stages here as needed
    }
}
