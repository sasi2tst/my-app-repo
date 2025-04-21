pipeline {
  agent {
    kubernetes {
      yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:20.10
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
'''
    }
  }
  environment {
    AWS_REGION = 'us-east-1'
    ECR_REPO = '857896345731.dkr.ecr.us-east-1.amazonaws.com/my-app'
    EKS_CLUSTER = 'my-eks-test-cluster'
  }
  stages {
    stage('Checkout') {
      steps {
        git credentialsId: 'github-credentials', url: 'https://github.com/sasi2tst/my-app-repo.git'
      }
    }
    stage('Build Docker Image') {
      steps {
        container('docker') {
          sh 'docker build -t ${ECR_REPO}:${BUILD_NUMBER} .'
        }
      }
    }
    stage('Push to ECR') {
      steps {
        container('docker') {
          sh '''
            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
            docker push ${ECR_REPO}:${BUILD_NUMBER}
            docker tag ${ECR_REPO}:${BUILD_NUMBER} ${ECR_REPO}:latest
            docker push ${ECR_REPO}:latest
          '''
        }
      }
    }
    stage('Deploy to EKS') {
      steps {
        container('kubectl') {
          sh '''
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}
            kubectl apply -f deployment.yaml
          '''
        }
      }
    }
  }
}
