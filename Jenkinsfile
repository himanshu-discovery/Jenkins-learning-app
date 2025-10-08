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
        echo 'Building... Diagnostic mode ON'
        sh '''
          echo "---- /etc/resolv.conf inside container ----"
          cat /etc/resolv.conf || true

          echo "---- Check DNS resolution via node dns.lookup ----"
          node -e "require('dns').lookup('registry.npmjs.org',(e,r)=>console.log('dns lookup:', e ? e.message : r))" || true

          echo "---- Try nslookup (install bind-tools if needed) ----"
          apk add --no-cache bind-tools > /dev/null 2>&1 || true
          nslookup registry.npmjs.org || true

          echo "---- Test HTTP connectivity to registry ----"
          wget -qO- --timeout=10 https://registry.npmjs.org/-/v1/search?size=1 || echo "wget-failed"

          echo "---- Run npm ci with retries (fallback to --no-audit) ----"
          set -o pipefail
          for i in 1 2 3; do
            echo "Attempt $i: npm ci"
            npm ci --loglevel=verbose 2>&1 | tee /tmp/npm-ci.log && break || sleep 5
          done

          if [ ! -f /tmp/npm-ci.log ] || grep -q "EAI_AGAIN" /tmp/npm-ci.log; then
            echo "npm ci failing with DNS issues, doing fallback: npm ci --no-audit"
            npm ci --no-audit --no-fund --loglevel=verbose 2>&1 | tee /tmp/npm-ci-noaudit.log || true
          fi

          echo "---- Build (if install succeeded) ----"
          npm run build || echo "npm run build failed (likely due to earlier errors)"

          echo "---- ls of workspace ----"
          ls -la
        '''
      }
            steps {
                echo 'Building...'
                sh '''
                 ping google.com
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