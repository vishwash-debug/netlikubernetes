pipeline {
  agent any

  environment {
    SONAR_HOME = tool "Sonar"
    IMAGE_NAME = "vishwashducker12/kbimg"
    IMAGE_TAG = "${BUILD_NUMBER}"
    DOCKER_CREDS = credentials('dockerhub-creds')
  }

  stages {

    stage('CODE') {
      steps {
        git url: "https://github.com/vishwash-debug/netlikubernetes.git", branch: "main"
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar') {
          sh """
            ${SONAR_HOME}/bin/sonar-scanner \
            -Dsonar.projectName=NETLIPROJ \
            -Dsonar.projectKey=NETLIPROJ
          """
        }
      }
    }

    stage('BUILD') {
      steps {
        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
      }
    }

    stage('IMAGE_CHECK') {
      steps {
        sh "trivy image --severity CRITICAL --exit-code 0 ${IMAGE_NAME}:${IMAGE_TAG}"
      }
    }

    stage('IMAGE_PUSH') {
      steps {
        sh """
          echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin
          docker push ${IMAGE_NAME}:${IMAGE_TAG}
        """
      }
    }

    stage('Deploy to k8s') {
      steps {
        withAWS(credentials: 'aws-creds') {
          sh """
            aws eks --region us-east-1 update-kubeconfig --name netlicluster

            sed -i 's|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml

            kubectl apply -f k8s/deployment.yaml
            kubectl apply -f k8s/service.yaml
          """
        }
      }
    }

  }
}
