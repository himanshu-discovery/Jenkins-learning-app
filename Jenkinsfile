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
                // run npm with log capturing
                sh '''
              
                   npm ci
                    echo "Running build..."
                    npm run build 2>&1 | tee npm-build.log
                    ls -la build || true
                '''
            }
        }
        stage('Debug Network') {
            steps {
                sh '''
            echo "===== 🌐 Network Debug Info ====="

            echo "🔹 Hostname:"
            hostname

            echo "\n🔹 Network Interfaces:"
            ip addr show || true

            echo "\n🔹 Default Gateway:"
            ip route show default || true

            echo "\n🔹 DNS Configuration (/etc/resolv.conf):"
            cat /etc/resolv.conf || true

            echo "\n🔹 Docker Network (via hostname inspection if Docker CLI available):"
            docker inspect $(hostname) \
                --format '{{json .NetworkSettings.Networks}}' \
                2>/dev/null || echo "Docker not accessible in this container"

            echo "\n🔹 Check IP Connectivity:"
            ping -c 3 8.8.8.8 || echo "❌ Cannot reach 8.8.8.8"

            echo "\n🔹 Check DNS Resolution:"
            nslookup google.com 2>/dev/null || echo "❌ DNS resolution failed"

            echo "\n🔹 Check HTTP Connection:"
            curl -I https://registry.npmjs.org/ || echo "❌ Cannot reach registry.npmjs.org"

            echo "=================================="
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
