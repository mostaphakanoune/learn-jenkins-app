pipeline {
    agent any
   
    stages {
        /*
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la 
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la 
                '''
            }
        }
        */
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
        
            steps {
                sh '''
                    npm ci
                    test -f build/index.html
                    npm test
                '''
            }
        }

         stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
        
            steps {
                sh '''
                    npm install serve
                    # The symbo '&' is for start the server in the background other wise the next command won't run
                    node_modules/.bin/serve -s build &
                    sleep 10 # wait for the server to start
                    # To get the report as html file you should add --reporter=html
                    npx playwright test --reporter=html
                '''
            }
        }

    }
    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
