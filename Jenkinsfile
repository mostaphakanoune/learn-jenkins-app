pipeline {
    agent any

    environment {
            NETLIFY_SITE_ID = 'fc322399-a9c3-4488-9ac0-60ad60c998f2'
            NETLIFY_AUTH_TOKEN = credentials('netlify-token')
            
    }
      
    stages {
        
        
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
        
        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                
                    steps {
                        sh '''
                            # This line is added to resolve some issues related to missing dependencies (react-scripts) in alpine image
                            npm install react-scripts
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                
                    steps {
                        sh '''
                            npm install serve
                            #The symbo '&' is for start the server in the background other wise the next command won't run
                            node_modules/.bin/serve -s build &
                            sleep 10 # wait for the server to start
                            # To get the report as html file you should add --reporter=html
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@latest --save-dev
                    node_modules/.bin/netlify --version  
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"  
                    node_modules/.bin/netlify status 
                    node_modules/.bin/netlify deploy --no-build --dir=build  # Deploy without running a build first
                '''
            }
        }
         stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@latest --save-dev
                    node_modules/.bin/netlify --version  
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"  
                    node_modules/.bin/netlify status 
                    node_modules/.bin/netlify deploy --no-build --dir=build --prod # Deploy without running a build first
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://papaya-caramel-0eee4f.netlify.app'
                 NETLIFY_SITE_ID = 'fc322399-a9c3-4488-9ac0-60ad60c998f2'
            }
        
            steps {
                sh '''
                    npx playwright test --reporter=html
                    echo "Testing Production Deployment: $CI_ENVIRONMENT_URL"  
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        } 
    }  
}
