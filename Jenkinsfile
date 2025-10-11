pipeline {
    agent any

    options {
        timeout(time: 45, unit: 'MINUTES')   // fail long-running builds
    }

    stages {
        stage('Prepare') {
            steps {
                echo "Workspace: ${env.WORKSPACE}"
                sh 'uname -a || true'
            }
        }
        stage('Build') {
            agent {
                    docker {
                        image 'node:24-alpine3.21'
                        reuseNode true
                    }
            }
            steps {
                sh '''
            apk add --no-cache iputils bind-tools || true
            echo "Container Hostname: $(hostname)"
            echo "Network Info:"
            ip addr show
            echo "DNS Config:"
            cat /etc/resolv.conf
            echo "Ping test:"
            ping -c 3 8.8.8.8 || true
        '''
            }
            steps {
                sh '''
                    echo "---- container /etc/resolv.conf ----"
                    cat /etc/resolv.conf || true
                    echo "---- dns lookup google ----"
                    if command -v nslookup >/dev/null 2>&1; then nslookup google.com || true; fi
                    echo "---- node & npm versions ----"
                    node --version || true
                    npm --version || true
                '''
                // run npm with log capturing
                sh '''
                    npm i react-scripts
                    set -o pipefail
                    echo "Running npm ci (logs -> npm-ci.log and npm-ci-noaudit.log)..."
                    npm ci --no-audit --no-fund --legacy-peer-deps --verbose 2>&1 | tee npm-ci.log || true
                    # If npm ci failed, also try with --no-audit as separate command and capture logs
                    if [ $? -ne 0 ]; then
                    npm ci --no-audit --no-fund --legacy-peer-deps 2>&1 | tee npm-ci-noaudit.log || true
                    fi

                    echo "Running build..."
                    npm run build 2>&1 | tee npm-build.log
                    ls -la build || true
                '''
            }
        }
    }
    post {
        always {
            echo 'Archiving logs and cleaning workspace'
            archiveArtifacts artifacts: 'npm-ci.log,npm-ci-noaudit.log', allowEmptyArchive: true
            cleanWs()
        }
        failure {
            echo 'Pipeline failed. Printing docker/container diagnostics (if available)...'
            sh '''
                echo "docker ps -a (host):" || true
                docker ps -a || true
                echo "docker network ls (host):" || true
                docker network ls || true
            '''
        }
    }
}
