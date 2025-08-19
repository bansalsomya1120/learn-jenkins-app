pipeline{
    agent any

    stages{
        stage("Build"){
            agent{
                docker{
                    image 'node-18:alpine'
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

        stage("Test"){
            agent {
                docker{
                    image 'node-18:alpine'
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

    }
}