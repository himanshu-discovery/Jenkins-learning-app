pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                    docker {
                        image 'node:24-alpine'
                        reuseNode true

                    }
                }
            steps {
                echo 'Building...'
                sh '''
                 ls -la
                 node --version
                 npm --version
                 npm ci --verbose
                 npm run build
                 ls -la
                 '''
            }
        }
    }
}