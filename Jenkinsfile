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
    }
}