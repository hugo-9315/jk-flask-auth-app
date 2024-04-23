pipeline {
  agent any
  
  environment {
    ID_DOCKER = "${ID_DOCKER_PARAMS}"
    DOCKERHUB_PASSWORD = "${PASS_DOCKER_PARAMS}"
    IMAGE_NAME = "jk-flask-auth-app"
    IMAGE_TAG = "latest"
    PORT_EXPOSED = ${PORT_EXPOSED}
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
            timeout /t 5
          '''
        }
      }
    }
    
    stage('Test image') {
        agent any
        steps {
            script {
                bat '''
                curl http://localhost:%PORT_EXPOSED% | findstr "Redirecting..."
                '''
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
  
  }

}
