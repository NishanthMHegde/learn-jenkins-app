pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh '''
                ls -la
                npm --version
                node --version
                npm ci
                npm run build
                ls -la
                '''
            }
        }
    }
}