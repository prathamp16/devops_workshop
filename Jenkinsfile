pipeline {
    agent any

    environment {
        JENKINS_NODE_COOKIE = 'dontKillMe'
        BUILD_ID = 'dontKillMe'
    }

    stages {
        stage('1. Checkout') {
            steps {
                echo '=================================================='
                echo 'STAGE 1: Checking out code from repository...'
                echo "Building commit: ${env.GIT_COMMIT ?: 'Local Commit'}"
                echo '=================================================='
            }
        }

        stage('2. Install & Build') {
            steps {
                echo '=================================================='
                echo 'STAGE 2: Installing dependencies and building...'
                echo '=================================================='
                sh '''
                    if ! command -v node &> /dev/null; then
                      if [ -d "$HOME/.nvm/versions/node" ]; then
                        LATEST_NVM_NODE=$(ls -d "$HOME/.nvm/versions/node/"* 2>/dev/null | tail -n 1)
                        if [ -n "$LATEST_NVM_NODE" ]; then
                          export PATH="$LATEST_NVM_NODE/bin:$PATH"
                        fi
                      fi
                    fi
                    export PATH="/usr/local/bin:/usr/bin:/bin:$PATH"
                    
                    npm install
                    npm run build
                '''
            }
        }

        stage('3. Automated Tests') {
            steps {
                echo '=================================================='
                echo 'STAGE 3: Running automated unit & API tests...'
                echo '=================================================='
                sh 'npm test'
            }
        }

        stage('4. Deploy to Staging (Port 3001)') {
            steps {
                echo 'STAGE 4: Deploying to STAGING...'
                sh 'chmod +x ./scripts/deploy.sh'
                sh './scripts/deploy.sh staging'
            }
        }

        
        

        stage('5. Smoke Test Staging') {
            steps {
                echo '=================================================='
                echo 'STAGE 5: Verifying Staging Deployment on Port 3001...'
                echo '=================================================='
                sh 'chmod +x ./scripts/health-check.sh'
                sh './scripts/health-check.sh 3001'
            }
        }

        stage('6. Deploy to Production (Port 3000)') {
            steps {
                echo 'STAGE 6: Promoting to PRODUCTION...'
                sh './scripts/deploy.sh production'
            }
        }
    }

    post {
        success {
            echo '=================================================='
            echo 'PIPELINE SUCCESSFUL!'
            echo 'Production UI: http://localhost:3000'
            echo 'Staging UI: http://localhost:3001'
            echo '=================================================='
        }
        failure {
            echo '=================================================='
            echo 'PIPELINE FAILED! Fix the breaking tests or build error and push again.'
            echo '=================================================='
        }
    }
}
