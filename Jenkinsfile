pipeline {
    agent any

    stages {
        /*
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '--user root'
                    reuseNode true
                }
            }
            steps{
                sh'''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls  -la
                '''
            }
        }
        */

pipeline {
    agent any
    stages {
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh """
                    node --version
                    npm --version
                    npm ci
                    npm test
                """
            }
        }
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.50.1-noble'
                    reuseNode true
                }
            }
            steps {
                sh """
                    node --version
                    npm --version
                    npm install -g serve
                    serve -s build &
                    sleep 5  # Give server time to start
                    npx playwright test
                """
            }
        }
    }
    post {
        always {
            sh 'mkdir -p test-results'
            sh 'touch test-results/junit.xml'
            junit 'test-results/junit.xml'
        }
    }
}
