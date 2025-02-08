pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '4c2a28cc-7eb9-4657-bfd4-828251cbfab8'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        CI_ENVIRONMENT_URL = 'https://67a3a52ac85bb108e666765b--dainty-llama-3e7693.netlify.app'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '--user root'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'small change'
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
                            args '--user root'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            args '--user root'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false, 
                                alwaysLinkToLastBuild: false, 
                                keepAll: false, 
                                reportDir: 'playwright-report', 
                                reportFiles: 'index.html', 
                                reportName: 'playwright Local Report', 
                                reportTitles: '', 
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '--user root'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli -g node-jq
                    netlify --version
                    echo "deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                '''
                script{
                    env.STAGING_URL = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
        }
               stage('Staging E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            args '--user root'
                            reuseNode true
                        }
                    }
                    environment {
                        CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
                    steps {
                        sh '''
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false, 
                                alwaysLinkToLastBuild: false, 
                                keepAll: false, 
                                reportDir: 'playwright-report', 
                                reportFiles: 'index.html', 
                                reportName: 'staging E2E Report', 
                                reportTitles: '', 
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am Sure!'
                }
            }
        }
        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '--user root'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli -g
                    netlify --version
                    echo "deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                '''
            }
        }
                stage('Prod E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            args '--user root'
                            reuseNode true
                        }
                    }
                    environment {
                        CI_ENVIRONMENT_URL = 'https://67a3a52ac85bb108e666765b--dainty-llama-3e7693.netlify.app'
                    }
                    steps {
                        sh '''
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false, 
                                alwaysLinkToLastBuild: false, 
                                keepAll: false, 
                                reportDir: 'playwright-report', 
                                reportFiles: 'index.html', 
                                reportName: 'prod E2E Report', 
                                reportTitles: '', 
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
    }
}
}