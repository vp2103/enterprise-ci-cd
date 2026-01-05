pipeline {
  agent any

  environment {
    IMAGE_NAME = "vp2103/enterprise-ci-cd"
    IMAGE_TAG  = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/vp2103/enterprise-ci-cd.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          docker build -t $IMAGE_NAME:$IMAGE_TAG .
        '''
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USERNAME',
          passwordVariable: 'DOCKER_PASSWORD'
        )]) {
          sh '''
            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            docker push $IMAGE_NAME:$IMAGE_TAG
          '''
        }
      }
    }

    stage('Deploy (Validation)') {
      steps {
        sh '''
          docker rm -f enterprise-ci || true
          docker run -d --name enterprise-ci -p 5000:5000 $IMAGE_NAME:$IMAGE_TAG
          sleep 5
          curl http://localhost:5000
        '''
      }
    }
  }
}
