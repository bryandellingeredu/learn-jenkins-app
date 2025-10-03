pipeline {
    agent any
    environment {
        // Docker Desktop on Windows host, reachable from Linux containers:
        DOCKER_HOST = 'tcp://host.docker.internal:2375'
    }
    stages {
        stage('diagnose') {
            steps {
                sh '''
                  echo "DOCKER_HOST=$DOCKER_HOST"
                  which docker || true
                  docker version
                '''
            }
        } // <-- this was missing
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

        stage('Test'){
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
        }

          stage('E2E'){
            agent {
                docker {
                   image 'mcr.microsoft.com/playwright:v1.55.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install -g server
                   serve -s build
                   npx playwright test
                '''
            }
        }
    }

    post{
       always{
        junit 'test-results/junit.xml'
       } 
    }
}
