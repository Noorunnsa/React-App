pipeline {
    agent any
    environment {
        // Define Docker Hub credentials ID stored in Jenkins credentials store
        DOCKERHUB_CREDENTIALS = "dockerHubCredentials"
        KUBECONFIG_CREDENTIALS_ID = "kubeconfig"
    }
    stages {
        stage('Checkout main branch') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'origin/main']], userRemoteConfigs: [[url: 'https://github.com/Noorunnsa/React-App.git']]])
            }
        }
        stage('SonarQube Analysis') {
         steps {
             script {
                 def scannerHome = tool 'SonarQubeScanner'
                 withSonarQubeEnv('sonarqube') {
                   sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Sonar_Jenkins -Dsonar.host.url=http://3.110.122.47:9000/ -Dsonar.login=sqp_9f184c9b6ac6e5e240ece3a186c091c5f8e56835"
                 }
               }
            }
        }
     stage("Quality Gate") {
       steps {
               timeout(time: 5, unit: 'MINUTES') {
                 waitForQualityGate abortPipeline: true
             }
         }
     }
        stage('Build Docker Image') {
            steps {
          sh "docker build -t noorunnisa/react-app:${BUILD_NUMBER} ."
          sh "docker images"
          sh "sleep 10"
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
                        sh "docker push ${DOCKERHUB_USERNAME}/react-app:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage('modify manifest tag') {
           steps {
              sh "sed -i 's|noorunnisa/react-app:[^ ]*|noorunnisa/react-app:${BUILD_NUMBER}|' ./manifests/deployment.yaml"
              sh 'cat ./manifests/deployment.yaml'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy to Kubernetes using manifests from the repository
                    withCredentials([file(credentialsId: KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG')]) {
                        sh 'kubectl apply -f ./manifests/deployment.yaml'
                        sh 'kubectl apply -f ./manifests/service.yaml'
                        sh 'kubectl apply -f ./manifests/ingress.yaml'
                    }
                }
            }
        }
    }
}
