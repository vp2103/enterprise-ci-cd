pipeline {
  agent any

  environment {
    IMAGE_NAME = "enterprise-ci-cd"
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/vp2103/enterprise-ci-cd.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
        }
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USERNAME',
          passwordVariable: 'DOCKER_PASSWORD'
        )]) {
          sh """
            docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}
            echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
            docker push ${DOCKER_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}
          """
        }
      }
    }

    stage('Deploy (Validation)') {
      steps {
        sh """
          docker rm -f enterprise-ci-cd || true
          docker run -d --name enterprise-ci-cd -p 5000:5000 ${DOCKER_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}
          sleep 5
          curl http://localhost:5000
        """
      }
    }
  }
}
