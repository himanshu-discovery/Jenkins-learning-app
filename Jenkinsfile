pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                    docker {
                        image 'node:24-alpine3.21'
                        reuseNode true
                        args '-p 3000:3000 --dns=8.8.8.8 --dns=8.8.4.4 --network=jenkins_network'
                    }
            }
            steps {
                sh '''
                echo "---- container /etc/resolv.conf ----"
                cat /etc/resolv.conf || true

                echo 'Building...'
                ls -la
                node --version
                npm --version
                if [ $? -ne 0 ]; then
                    npm ci --no-audit --no-fund --legacy-peer-deps  --verbose || true
                fi
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
