pipeline {
    agent { docker { image 'ruby:3.0.3-alpine' } }
    stages {
        stage('build') {
            steps {
                sh 'ruby --version'
            }
        }
    }
}