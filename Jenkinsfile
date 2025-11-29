pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '195409a8-4b5a-4877-b672-a89b9de38cc1'
        NETLIFY_AUTH_TOKEN = credentials('netlfy-token')
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-bullseye'   // FIX: alpine → bullseye
                    reuseNode true
                }
            }
            steps {
                sh '''
                    apt-get update
                    apt-get install -y bash   # FIX for ENOENT bash error

                    ls -la
                    node --version
                    npm --version

                    npm ci
                    npm run build

                    ls -la build
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-bullseye'   // FIX: alpine → bullseye
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            apt-get update
                            apt-get install -y bash

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
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright Local'
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-bullseye'   // FIX
                    reuseNode true
                }
            }
            steps {
                sh '''
                    apt-get update
                    apt-get install -y bash

                    npm install netlify-cli

                    echo "Deploying to staging..."
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --site=$NETLIFY_SITE_ID
                '''
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-bullseye'   // FIX
                    reuseNode true
                }
            }
            steps {
                sh '''
                    apt-get update
                    apt-get install -y bash

                    npm install netlify-cli

                    echo "Deploying to production..."
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod --site=$NETLIFY_SITE_ID
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
                CI_ENVIRONMENT_URL = 'https://unrivaled-profiterole-12edf0.netlify.app/'
            }

            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright E2E'
                    ])
                }
            }
        }
    }
}
