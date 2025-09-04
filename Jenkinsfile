pipeline{
    agent any

    environment{
        NETLIFY_SITE_ID = 'e8de2b71-78fd-4281-b959-207a7e9d779d'
        NETLIFY_AUTH_TOKEN = credentials('Netlify_Token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages{
        stage("Build"){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                /*
                npm ci is used to connect to ci/cd servers
                It will create 'node-modules' folder which will never be, shouldn't be part of original source git repo.

                npm run build is used to run the build in the container. 
                It will create 'build' folder which will never be, shouldn't be part of original source git repo.
                */

                sh '''
                    ls -al
                    npm --version
                    node --version
                    npm ci
                    npm run build
                    ls -al
                '''
            }

        }

        stage("Tests"){
            /* Executing Unit Test and E2E Test stages parallely as it will save time.
            We focus on stages to run parallely which have no dependency on each other, 
            their execution time difference must be less, also to ensure "fail fast". 
            All these things are guidelines for efficient parallel stage execution.
            */
            parallel{
                stage("Unit Test"){
                    agent {
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                }

                stage("E2E Test"){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }

                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test
                        '''
                    }
                }

            }
        }

        stage("Deploy staging"){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to prodcution.Site id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --json > deploy_output.json
                    
                '''
                script{
                    env.STAGING_URL = sh(script :"node_modules/.bin/node-jq -r '.deploy_url' deploy_output.json", returnStdout : true )
                }
            }

        }

        stage("Staging E2E"){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment{
                        CI_ENVIRONMENT_URL = "$env.STAGING_URL"
                    }

                    steps{
                        sh '''
                            npx playwright test
                        '''
                    }
                   /* post{
                        always{
                            publishHTML([allowmissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod report', reportTitles: '', userWrappedFileDirectory: true])
                        }
                    }*/
            }

    /*
        stage('Approval') {
            steps {
                timeout(time: 30, unit: 'SECONDS') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
                
            }
        } */

        stage("Deploy PROD"){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment{
                        CI_ENVIRONMENT_URL = 'https://jazzy-belekoy-a627dd.netlify.app'
                    }

                    steps{
                        sh '''
                            node --version
                            npm install netlify-cli@20.1.1
                            node_modules/.bin/netlify --version
                            echo "Deploying to prodcution.Site id: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --prod
                            npx playwright test
                        '''
                    }
                   /* post{
                        always{
                            publishHTML([allowmissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod report', reportTitles: '', userWrappedFileDirectory: true])
                        }
                    }*/
            }
        
    }
}
