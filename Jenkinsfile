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
                image: jenkins/inbound-agent:4.11.2-4
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
                - name: JAVA_OPTS
                  value: "-Xmx512m"
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
                image: bitnami/kubectl:1.24
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
                    sh 'whoami'
                    sh 'pwd'
                    sh 'ls -l /home/jenkins/agent/workspace'
                    checkout scm
                    sh 'ls -la /home/jenkins/agent/workspace/my-app-pipeline'
                    sh 'cat /home/jenkins/agent/workspace/my-app-pipeline/Dockerfile || echo "Dockerfile not found"'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        sh 'whoami'
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'docker --version'
                        sh 'docker info || echo "Docker daemon not running"'
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