pipeline {
    agent any
    environment {
          SSH_HOST = 'ubuntu@192.168.49.1'
      }
    stages {
        stage('Checkout GitHub repo') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/sauvikdevops/dockerdemo']])
            }
        }
        
        stage('Build and Tag Docker Image') {
            steps {
                script {
                    sh 'docker build . -t hellodocker'
                    sh 'docker tag hellodocker sauvikdevops/learning'
                }
            }
        }
        
        stage('Push the Docker Image to DockerHUb') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_hub', variable: 'docker_hub')]) {
                    sh 'docker login -u sauvik.devops@gmail.com -p ${docker_hub}'
} 
                    sh 'docker push sauvikdevops/learning'
                }
            }
        }
        
        stage('Deploy to Kubernetes'){
              steps {
                  sshagent(['k8-connect-ssh-credential']) {
                      // Inside this block, you have SSH authentication set up

                      // Check if deployment already exists
                      script {
                          def deploymentExists = sh(script: "ssh ${env.SSH_HOST} 'kubectl get deployment cicd-learning --no-headers --ignore-not-found'", returnStatus: true) == 0

                          // If deployment exists, delete it
                          if (deploymentExists) {
                              sh "ssh ${env.SSH_HOST} 'kubectl delete deployment cicd-learning'"
                          }
                      }

                      // Deploy Kubernetes manifest
                      sh "ssh ${env.SSH_HOST} 'kubectl create deployment cicd-learning --image=docker.io/sauvikdevops/learning:latest'"

                      // Check if service already exists
                      script {
                          def serviceExists = sh(script: "ssh ${env.SSH_HOST} 'kubectl get service cicd-learning --no-headers --ignore-not-found'", returnStatus: true) == 0

                          // If service doesn't exist, expose it
                          if (!serviceExists) {
                              sh "ssh ${env.SSH_HOST} 'kubectl expose deployment cicd-learning --type=NodePort --port=8090'"
                          }
                      }

                      // Access the app and get the URL
                          script {
                              def serviceUrl = sh(script: "ssh ${env.SSH_HOST} 'minikube service cicd-learning --url'", returnStdout: true).trim()

                      // Print the URL to Jenkins build logs
                              echo "Application URL: ${serviceUrl}"
                      }
                  }
              }
          }
    }
}