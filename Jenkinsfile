pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '2b262f45-2eb0-49c5-a8e4-e647189c7fe5'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        // stage('Build') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //         ls -la
        //         npm --version
        //         node --version
        //         npm cache clean --force
        //         npm ci
        //         npm run build
        //         ls -la
        //         '''
        //     }
        // }

        stage('Run Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        echo "Test Stage"
                        test -f build/index.html
                        npm test
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }
                stage('E2ETest') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        echo "E2E Test Stage"
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        rm -rf playwright-report
                        npx playwright test --reporter=list,html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report'
                            ])
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '--network=host'
                }
            }
            steps {
                sh '''
                echo 'Starting the Deployment stage'
                npm install netlify-cli
                ./node_modules/.bin/netlify status
                ./node_modules/.bin/netlify deploy --prod --dir=./build
                '''
            }
        }
        stage('Deploy to staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '--network=host'
                }
            }
            steps {
                sh '''
                echo 'Starting the Deployment stage to deploy to staging'
                npm install netlify-cli
                ./node_modules/.bin/netlify status
                ./node_modules/.bin/netlify deploy --dir=./build
                '''
            }
        }
        stage('ProdE2ETest') {
            environment {
                CI_ENVIRONMENT_URL = 'https://melodic-manatee-f502a0.netlify.app'
            }

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "E2E Prod Test Stage"
                npx playwright test --reporter=list,html
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright Prodt test HTML Report'
                    ])
                }
            }
        }
        
    }
   
}