pipeline {
    agent any

    stages {
        stage('Build') {
            script:
                - unset DOCKER_HOST
                - docker login $CI_REGISTRY -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD
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
    }
}
