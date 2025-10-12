pipeline {
    agent any

    options {
        timeout(time: 45, unit: 'MINUTES')   // fail long-running builds
    }

    stages {
        // This is a build Stage container commented out for now
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
        stage('Test') {
             agent {
                    docker {
                        image 'node:24-alpine3.21'
                        reuseNode true
                    }
            }
            steps {
                echo "Test stage"

                    sh '''
                        echo "🔍 Checking if build/index.html exists..."

                        if [ -f build/index.html ]; then
                            echo "✅ File build/index.html exists."
                        else
                            echo "❌ File build/index.html does NOT exist!"
                            exit 1
                        fi

                        npm run test
                    '''


            }
        }
         stage('E2E Test') {
             agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.56.0-noble'
                        reuseNode true
                        args ''
                    }
            }
            steps {
                echo "Test stage"

                    sh '''
                       npm install serve
                       node_modules/.bin/serve -s build & 
                       sleep 10
                       npx playwright test
                    '''
            }
        }
    }
    post {
        always {
            junit 'test-results/junit.xml'
            echo 'Archiving logs and cleaning workspace'
            archiveArtifacts artifacts: 'npm-ci.log,npm-ci-noaudit.log', allowEmptyArchive: true

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
