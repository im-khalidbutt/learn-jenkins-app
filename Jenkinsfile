pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'ea85029a-9755-446b-92f2-0fdde8017bb0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage('Docker'){
            steps {
                sh 'docker build -t my-playwright .'
            }
        }

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

        stage ('Tests'){
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        /* stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
               sh '''
                npm install netlify-cli node-jq
                node_modules/.bin/netlify --version
                echo "deploying to prod site id: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --no-build --json > deploy-output.json
               ''' 
                script {
                    env.STAGING_URL = sh(script:"node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
        } */


        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "STAGING_URL_TO_SETS"
            }
            steps {
            sh '''
                netlify --version
                echo "deploying to prod site id: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --no-build --json > deploy-output.json
                CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                npx playwright test --reporter=html
            ''' 
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E staging', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }


        /* stage('Approval') {
            steps {
                timeout(time: 15, unit: 'HOURS') {
                    input message: 'Are you sure want to deploy?', ok: 'Yes I am Sure..!!'
                }
            }
        } */

        /* stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
               sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "deploying to prod site id: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --prod --dir=build --site=$NETLIFY_SITE_ID --no-build
               ''' 
            }
        } */

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://subtle-dango-185c45.netlify.app'
            }
            steps {
            sh '''
                node --version
                // npm install netlify-cli
                netlify --version
                echo "deploying to prod site id: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --prod --dir=build --site=$NETLIFY_SITE_ID --no-build
                npx playwright test --reporter=html
            ''' 
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E prod', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
