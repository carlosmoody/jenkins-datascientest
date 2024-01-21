pipeline {
  agent any
  stages {
    stage('Docker Build') {
      parallel {
        stage('Movie Service') {
          steps {
            script {
              sh '''
docker rm -f jenkins-movie
cd movie-service
docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG .
sleep 6
'''
            }

          }
        }

        stage('Cast Service') {
          steps {
            script {
              sh '''
docker rm -f jenkins-cast
cd cast-service
docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG .
sleep 6
'''
            }

          }
        }

      }
    }

    stage('Docker run') {
      parallel {
        stage('Movie Service') {
          steps {
            script {
              sh '''
docker run -d -p 8001:8000 --name jenkins-movie $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
sleep 10
'''
            }

          }
        }

        stage('Cast Service') {
          steps {
            script {
              sh '''
docker run -d -p 8002:8000 --name jenkins-cast $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
sleep 10
'''
            }

          }
        }

      }
    }

    stage('Test Acceptance') {
      parallel {
        stage('Movie Service') {
          steps {
            script {
              sh '''
curl localhost:8001
'''
            }

          }
        }

        stage('Cast Service') {
          steps {
            script {
              sh '''
curl localhost:8002
'''
            }

          }
        }

      }
    }

    stage('Docker Push') {
      parallel {
        stage('Docker Push') {
          environment {
            DOCKER_PASS = credentials('DOCKER_HUB_PASS')
          }
          steps {
            script {
              sh '''
docker login -u $DOCKER_ID -p $DOCKER_PASS
docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
'''
            }

          }
        }

        stage('error') {
          steps {
            script {
              sh '''
docker login -u $DOCKER_ID -p $DOCKER_PASS
docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
'''
            }

          }
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
cp fastapi/values.yaml values.yml
cat values.yml
sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
helm upgrade --install app fastapi --values=values.yml --namespace dev
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
cp fastapi/values.yaml values.yml
cat values.yml
sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
helm upgrade --install app fastapi --values=values.yml --namespace staging
'''
        }

      }
    }

    stage('Deploiement en prod') {
      environment {
        KUBECONFIG = credentials('config')
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
cp fastapi/values.yaml values.yml
cat values.yml
sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
helm upgrade --install app fastapi --values=values.yml --namespace prod
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