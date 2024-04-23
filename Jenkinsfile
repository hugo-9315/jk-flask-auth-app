pipeline {
  agent any
  
  environment {
    ID_DOCKER = "${ID_DOCKER_PARAMS}"
    DOCKERHUB_PASSWORD = "${PASS_DOCKER_PARAMS}"
    IMAGE_NAME = "jk-flask-auth-app"
    IMAGE_TAG = "latest"
    PORT_EXPOSED = "${PORT_EXPOSED}"
  }

  stages {
    stage('Build image') {
      agent any
      steps {
        script {
          bat 'docker build -t %ID_DOCKER%/%IMAGE_NAME%:%IMAGE_TAG% .'
        }
      }
    }
    
    stage('Run container based on builded image') {
      agent any
      steps {
        script {
          bat '''
            echo "Clean Environment"
            docker rm -f %IMAGE_NAME% || echo "container does not exist"
            docker run --name %IMAGE_NAME% -d -p %PORT_EXPOSED%:5000 -e PORT=5000 %ID_DOCKER%/%IMAGE_NAME%:%IMAGE_TAG%
          '''
        }
      }
    }
    
    stage('Test image') {
            steps {
                script {
                    echo "Exécution des tests"
                }
            }
        }
    
    stage('Clean Container') {
      agent any
      steps {
        script {
          bat '''
            docker stop %IMAGE_NAME%
            docker rm %IMAGE_NAME%
          '''
          }
      }
    }
    
    stage('Login and Push Image on docker hub') {
            steps {
                script {
                    bat '''
                        docker login -u %ID_DOCKER% -p %DOCKER_PASSWORD%
                        docker push %ID_DOCKER%/%IMAGE_NAME%:%IMAGE_TAG%
                    '''
                }
            }
        }

    stage('Push image in staging and deploy it') {
      when {
        expression { GIT_BRANCH == 'origin/main' }
      }
      agent any
      environment {
        RENDER_STAGING_DEPLOY_HOOK = credentials('render_flask_key')
      }  
      steps {
        script {
          bat '''
            echo "Staging"
            echo %RENDER_STAGING_DEPLOY_HOOK%
            curl %RENDER_STAGING_DEPLOY_HOOK%
            '''
          }
      }
    }
  
  }

  post {
        success {
            mail to: 'marchaohugo@gmail.com',
                 subject: "Succès du Pipeline ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "Le pipeline a réussi. L'application a été déployée sur Render."
        }
        failure {
            mail to: 'marchaohugo@gmail.com',
                 subject: "Échec du Pipeline ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "Le pipeline a échoué. Veuillez vérifier Jenkins pour plus de détails."
        }
    }

}
