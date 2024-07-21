pipeline {
    agent any
    environment {
        // Define Docker Hub credentials ID stored in Jenkins credentials store
        DOCKERHUB_CREDENTIALS = "dockerHubCredentials"
        KUBE_CONFIG = "kubeconfig"
    }
    stages {
        stage('Checkout main branch') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'origin/main']], userRemoteConfigs: [[url: 'https://github.com/Noorunnsa/React-App.git']]])
            }
        }
        stage('Build Docker Image') {
            steps {
          sh "docker build -t noorunnisa/react-app:${BUILD_NUMBER} ."
          sh "docker images"
          sh "sleep 10"
            }
        }
        stage('Test Docker Image') {
      steps {
          sh "docker run -d -p 8080:80 --name react-app noorunnisa/react-app:${BUILD_NUMBER}"
          sh "sleep 10"
          sh "docker rm -f react-app"
          sh "echo testing complete"
        }
      }
      stage('Login to DockerHub and Push Docker Image') {
            steps {
                script {
                    // Load Docker Hub credentials from Jenkins credentials store
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKERHUB_USERNAME', 
passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        // Login to Docker Hub
                        sh "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"
                        // Push Docker image
                        sh "docker push index.docker.io/${DOCKERHUB_USERNAME}/react-app:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage('modify manifest tag') {
           steps {
            // sh "sed -i -e 's%noorunnisa/react-app:.*%noorunnisa/react-app:${BUILD_NUMBER}%g' manifests/deployment.yml"
             // sh "sed -i "s|noorunnisa/react-app:[^ ]*|noorunnisa/react-app:${BUILD_NUMBER}|" manifests/deployment.yaml"
              sh "sed -i 's/noorunnisa/react-app:[^ ]*/noorunnisa/react-app:${BUILD_NUMBER}/' manifests/deployment.yaml"
              sh 'cat manifests/deployment.yml'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy to Kubernetes using manifests from the repository
                    withCredentials([credentialsId: 'kubeconfig']) {
                        sh 'kubectl apply -f deployment.yaml'
                    }
                }
            }
        }
    }
