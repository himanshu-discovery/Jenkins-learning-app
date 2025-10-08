pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                    docker {
                        image 'node:24-alpine'
                        reuseNode true
                        args '-p 3000:3000 --dns=8.8.8.8 --dns=8.8.4.4'
                    }
            }
            steps {
                echo 'Building...'
                sh '''
                 musl google.com
                 ls -la
                 node --version
                 npm --version
                 npm ci --no-audit --no-fund --loglevel=verbose
                 npm run build
                 ls -la
                 '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'npm-ci.log,npm-ci-noaudit.log', allowEmptyArchive: true
            cleanWs()
        }
    }
}
