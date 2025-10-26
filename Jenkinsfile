pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '61fac6c4-c155-45a9-8754-debbd0850a81'
        NETLIFY_AUTH_TOKEN = credentials('netlify_deploy_token')
        REACT_APP_VERSION = "1.0.${env.BUILD_ID}"
    }

    stages {

        stage('Docker') {
            steps {
                sh 'docker build -t learn-jenkins-playwright .'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'learn-jenkins-playwright'
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
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'learn-jenkins-playwright'
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
                            image 'learn-jenkins-playwright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E test', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging plus E2E') {
            agent {
                docker {
                    image 'learn-jenkins-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy_stage.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy_stage.json)
                    npx playwright test  --reporter=html
                    rm deploy_stage.json
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E stage', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy prod plus E2E') {
            agent {
                docker {
                    image 'learn-jenkins-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://elaborate-tanuki-3ffac3.netlify.app'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E prod', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
