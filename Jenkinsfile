pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: jenkins-agent
                image: jenkins/inbound-agent:latest
                resources:
                  requests:
                    cpu: "500m"
                    memory: "512Mi"
                  limits:
                    cpu: "1000m"
                    memory: "1024Mi"
                command:
                - cat
                tty: true
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
        ECR_REGISTRY = '857896345731.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'my-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
                container('jenkins-agent') {
                    sh 'ls -l /home/jenkins/agent/workspace/my-app-pipeline'
                }
            }
        }    
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        def app = docker.build("${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}")
                    }
                }
            }
        }
        stage('Push to ECR') {
            steps {
                container('docker') {
                    script {
                        sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                        sh "docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
                    }
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                container('kubectl') {
                    script {
                        sh "kubectl apply -f deployment.yaml"
                    }
                }
            }
        }
    }
}