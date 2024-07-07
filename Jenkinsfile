pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c0b6c360-73af-4d94-8eaa-bcc029ad5bfc'
        NETLIFY_AUTH_TOKEN = credentials('Netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage('Docker'){
            steps{
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
        

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    // post {
                    //     always {
                    //         junit 'jest-results/junit.xml'
                    //     }
                    // }
                }

                stage('Prod E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    environment {
                        CI_DEPLOYMENT_SITE = 'https://.......'
                    }

                    steps {
                        // sh '''
                        //     npm install serve
                        //     node_modules/.bin/serve -s build &
                        //     sleep 10
                        //     npx playwright test  --reporter=html
                        // '''
                        echo "E2E testing ..."
                    }

                    // post {
                    //     always {
                    //         publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                    //     }
                    // }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_DEPLOYMENT_SITE = 'STAGING_URL'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID : $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(15) {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production. Site ID : $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
        }
    }
}
