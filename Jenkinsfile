pipeline {
  agent any
  stages {
    stage('Docker Build') {
      parallel {
        stage('Movie Service') {
          steps {
            script {
              sh '''
docker rm -f jenkins-movie || true
docker rmi $DOCKER_ID/$DOCKER_MOVIE_IMAGE:latest || true
cd movie-service
docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG .
docker tag $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG $DOCKER_ID/$DOCKER_MOVIE_IMAGE:latest
sleep 6
'''
            }

          }
        }

        stage('Cast Service') {
          steps {
            script {
              sh '''
docker rm -f jenkins-cast || true
docker rmi $DOCKER_ID/$DOCKER_CAST_IMAGE:latest || true
cd cast-service
docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG .
docker tag $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG $DOCKER_ID/$DOCKER_CAST_IMAGE:latest
sleep 6
'''
            }

          }
        }

      }
    }

    stage('Compose up') {
      steps {
        script {
          sh '''
docker compose up -d
sleep 10
'''
        }

      }
    }

    stage('Test acceptance') {
      parallel {
        stage('Movie Service') {
          steps {
            script {
              sh '''
curl localhost:8088/api/v1/movies/docs
'''
            }

          }
        }

        stage('Cast Service') {
          steps {
            script {
              sh '''
curl localhost:8088/api/v1/cast/docs
'''
            }

          }
        }

      }
    }

    stage('Docker Push') {
      environment {
        DOCKER_PASS = credentials('DOCKER_HUB_PASS')
      }
      steps {
        script {
          sh '''
docker login -u $DOCKER_ID -p $DOCKER_PASS
docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:latest
'''
        }

      }
    }

    stage('Deploiement en dev') {
      environment {
        KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
rm -Rf .kube
mkdir .kube
ls
cat $KUBECONFIG > .kube/config
cd jenkins-examen
helm upgrade --install app examen --values=values.yml --values=dev-values.yaml --namespace dev
'''
        }

      }
    }

    stage('Deploiement en qa') {
      environment {
        KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
rm -Rf .kube
mkdir .kube
ls
cat $KUBECONFIG > .kube/config
cd jenkins-examen
helm upgrade --install app examen --values=values.yml --values=qa-values.yaml --namespace qa
'''
        }

      }
    }

    stage('Deploiement en staging') {
      environment {
        KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
rm -Rf .kube
mkdir .kube
ls
cat $KUBECONFIG > .kube/config
cd jenkins-examen
helm upgrade --install app examen --values=values.yml --values=staging-values.yaml --namespace staging
'''
        }

      }
    }

    stage('Deploiement en prod') {
      environment {
        KUBECONFIG = credentials('config')
      }
      when {
                branch 'main'
        }
      steps {
        timeout(time: 15, unit: 'MINUTES') {
          input(message: 'Do you want to deploy in production ?', ok: 'Yes')
        }

        script {
          sh '''
rm -Rf .kube
mkdir .kube
ls
cat $KUBECONFIG > .kube/config
cd jenkins-examen
helm upgrade --install app examen --values=values.yml --values=prod-values.yaml --namespace prod
'''
        }

      }
    }

  }
  environment {
    DOCKER_ID = 'carlosmoody'
    DOCKER_MOVIE_IMAGE = 'movieservice'
    DOCKER_CAST_IMAGE = 'castservice'
    DOCKER_TAG = "${BUILD_ID}.0"
  }
}
