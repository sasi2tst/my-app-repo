pipeline {
    agent {
        kubernetes {
            defaultContainer 'jenkins-agent'
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                jenkins: slave
              namespace: jenkins
            spec:
              tolerations:
              - key: "app_type"
                operator: "Equal"
                value: "alpha"
                effect: "NoSchedule"
              containers:
              - name: jenkins-agent
                image: jenkins/inbound-agent:latest
                resources:
                  requests:
                    cpu: "200m"
                    memory: "256Mi"
                  limits:
                    cpu: "400m"
                    memory: "512Mi"
                command:
                - cat
                tty: true
                env:
                - name: JENKINS_URL
                  value: "http://jenkins.jenkins.svc.cluster.local:8080"
              - name: docker
                image: docker:20.10-dind
                command:
                - /usr/local/bin/dockerd-entrypoint.sh
                securityContext:
                  privileged: true
                resources:
                  requests:
                    cpu: "200m"
                    memory: "256Mi"
                  limits:
                    cpu: "400m"
                    memory: "512Mi"
              - name: kubectl
                image: bitnami/kubectl:latest
                command:
                - cat
                tty: true
                resources:
                  requests:
                    cpu: "200m"
                    memory: "256Mi"
                  limits:
                    cpu: "400m"
                    memory: "512Mi"
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
                container('jenkins-agent') {
                    checkout scm
                    sh 'pwd'
                    sh 'ls -l /home/jenkins/agent/workspace/my-app-pipeline'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        sh 'pwd'
                        sh 'ls -l'
                        sh 'docker --version'
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