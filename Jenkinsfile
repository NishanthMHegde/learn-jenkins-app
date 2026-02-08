pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '2b262f45-2eb0-49c5-a8e4-e647189c7fe5'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = '1.0.${BUILD_ID}'
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
                npm --version
                node --version
                npm cache clean --force
                npm ci
                npm run build
                ls -la
                '''
            }
        }

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
                            image 'my_playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        echo "E2E Test Stage"
                        serve -s build &
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
        
        stage('DeployStagingAndTest') {
            environment {
                CI_ENVIRONMENT_URL = "STAGING_URL_UNASSIGNED"
            }

            agent {
                docker {
                    image 'my_playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo 'Starting the Deployment stage to deploy to staging'
                netlify status
                netlify deploy --dir=./build --json > deploy_output.json
                echo "E2E Staging Test Stage"
                CI_ENVIRONMENT_URL=$(./node_modules/.bin/node-jq -r '.deploy_url' 'deploy_output.json')
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
                        reportName: 'Playwright Staging test HTML Report'
                    ])
                }
            }
        }
        stage('DeployToProdAndTest') {
            environment {
                CI_ENVIRONMENT_URL = 'https://melodic-manatee-f502a0.netlify.app'
            }

            agent {
                docker {
                    image 'my_playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo 'Starting the Deployment stage'
                netlify status
                netlify deploy --prod --dir=./build
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