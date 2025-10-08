pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                    docker {
                        image 'node:24-alpine3.21'
                        reuseNode true
                        args '-p 3000:3000 --dns=8.8.8.8 --dns=8.8.4.4'
                    }
            }
            steps {
                sh '''
                echo "---- container /etc/resolv.conf ----"
                cat /etc/resolv.conf || true

                tail -n 200 npm-ci*.log || true

                echo 'Building...'
                ls -la
                node --version
                npm --version
                npm ci --no-audit --no-fund --verbose
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
