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

        stage('E2E') {
            agent {
                docker {
                image 'node:18-bullseye'        // Debian-based, not Alpine
                args '-u root'                  // <-- run as root so --with-deps can apt-get
                reuseNode true
                }
            }
            environment {
                PLAYWRIGHT_JUNIT_OUTPUT = 'test-results/junit.xml'
            }
            steps {
                sh '''
                set -e

                # basics sometimes missing in slim images
                apt-get update -y
                apt-get install -y curl

                node --version
                npm --version

                npm ci
                npx playwright install --with-deps

                # serve build in background
                nohup npx serve -s build -l 3000 >/dev/null 2>&1 &

                # wait for server
                for i in $(seq 1 30); do
                    curl -sf http://127.0.0.1:3000 >/dev/null && break
                    sleep 1
                done

                npx playwright test --reporter=junit,line
                '''
            }
        }
    }

    post{
       always{
        junit 'jest-results/junit.xml'
       } 
    }
}
