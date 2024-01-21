pipeline {
  agent any
  stages {
    stage('Docker build Movie service') {
      steps {
        sh '''docker rm -f jenkins
           '''
        sh 'docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .'
      }
    }

  }
}