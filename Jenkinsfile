pipeline {
  agent none
  environment {
    IMAGE_NAME= "alpinehelloworld"
    IMAGE_TAG= "latest"
    STAGING_ENV= "alpine-lab-vm-staging"
    PRODUCTION_ENV= "alpine-lab-vm-production"
  }
  stages {
    stage('Build image') {
      agent any
      steps {
       script {
         sh 'docker build -t imane/${IMAGE_NAME}:${IMAGE_TAG} .'
       }
      } 
    }
    stage('Create a container based on the builded image') {
      agent any
      steps {
       script {
         sh '''
              docker rm -f ${IMAGE_NAME}
              docker run --name ${IMAGE_NAME} -d -p 80:5000 -e PORT=5000 imane/${IMAGE_NAME}:${IMAGE_TAG}
              sleep 5
         '''
       }
      }
    }
    stage('Test image') {
      agent any
      steps {
       script {
         sh '''
             curl http://172.17.0.1:80 | grep -q "Hello world!"
         '''
       }
      }
    }
    stage('Delete test container') {
      agent any
      steps {
       script {
         sh '''
              docker stop ${IMAGE_NAME} 
              docker rm ${IMAGE_NAME}
         '''
       }
      }
    }
     stage('Staging deployment') {
      when {
         expression { GIT_BRANCH == 'origin/master' }
      }
      agent any 
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
       script {
         sh '''
             heroku container:login
             heroku create $STAGING_ENV || echo 'project already exists'
             heroku container:push -a $STAGING_ENV web
             heroku container:release -a $STAGING_ENV web
         '''
       }
      }
    }
    stage('Production deployment') {
      when {
         expression { GIT_BRANCH == 'origin/master' }
      }
      agent any 
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
       script {
         sh '''
             heroku container:login
             heroku create $PRODUCTION_ENV || echo 'project already exists'
             heroku container:push -a $PRODUCTION_ENV web
             heroku container:release -a $PRODUCTION_ENV web
         '''
       }
      }
    }
  }
}
