pipeline {
    agent any

    environment {
        IMAGE_NAME = "varun210300/enterprise-ci-cd"
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
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy (Validation)') {
            steps {
                sh """
                docker rm -f enterprise-ci || true

                docker run -d --name enterprise-ci -p 5000:5000 varun210300/enterprise-ci-cd:${BUILD_NUMBER}

                sleep 5

                docker exec enterprise-ci python - <<EOF
                import urllib.request
                print(urllib.request.urlopen("http://localhost:5000").read().decode())
                EOF

                """
            }
        }
    }
}
