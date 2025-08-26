pipeline{
    agent any

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
        stage("Deploy"){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify
                    netlify --version
                '''
            }

        }
        
    }
}