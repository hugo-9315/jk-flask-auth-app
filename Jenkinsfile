/* import shared library */
@Library('shared-library')_

pipeline {
  agent any
  
  environment {
    ID_DOCKER = "${ID_DOCKER_PARAMS}"
    IMAGE_NAME = "jk-flask-auth-app"
    IMAGE_TAG = "latest"
  }

  stages {
    stage('Build image') {
      agent any
      steps {
        script {
          sh 'docker build -t ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG .'
        }
      }
    }
    
    stage('Run container based on builded image') {
      agent any
      steps {
        script {
          sh '''
            echo "Clean Environment"
            docker rm -f $IMAGE_NAME || echo "container does not exist"
            docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
            sleep 5
          '''
        }
      }
    }
    
    stage('Test image') {
      agent {
        docker {
          image 'python:3.8-slim'
          args '-v $PWD:/app'
        }
      }
      steps {
        // Run tests inside the Docker container
        sh '''
            pip install --no-cache-dir -r requirements.txt
            pytest
        '''
      }
    }
    
    stage('Clean Container') {
      agent any
      steps {
        script {
          sh '''
            docker stop $IMAGE_NAME
            docker rm $IMAGE_NAME
          '''
          }
      }
    }
    
    stage ('Login and Push Image on docker hub') {
      agent any
      environment {
        DOCKERHUB_PASSWORD  = credentials('159e35f1-8092-4d4b-bd1a-d66088a6d6e0')
      }            
      steps {
        script {
          sh '''docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG'''
        }
      }
    }
    
    stage('Push image in staging and deploy it') {
      when {
        expression { GIT_BRANCH == 'origin/main' }
      }
      agent any
      environment {
        RENDER_STAGING_DEPLOY_HOOK = credentials('render_karma_key')
      }  
      steps {
        script {
          sh '''
            echo "Staging"
            echo $RENDER_STAGING_DEPLOY_HOOK
            curl $RENDER_STAGING_DEPLOY_HOOK
            '''
          }
      }
    }
  }
  
  /* post {
    always {
      script {
        emailNotifier currentBuild.result
      }
    }  
  } */
}
