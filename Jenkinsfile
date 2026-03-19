pipeline {
  agent any

  stages {
    stage('CODE') {
      steps {
        git url:"", branch:"main"
      }
    }
    
    stage('EKS CLUSTER DEPLOY'){
      steps {
        sh "aws eks --region us-east-1 update-kubeconfig --name netlicluster "
        sh " kubectl get pods "
      }
    }
  }
}
