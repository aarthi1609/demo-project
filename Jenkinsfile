pipeline {
    agent any
        environment {
        GIT_CREDENTIALS_ID = 'github'
        SONARQUBE_SERVER = 'sonarqube'
        SONARQUBE_CREDENTIALS = 'sonar'
        DOCKERHUB_CREDENTIALS = '5f52c34c-dc54-453a-bdeb-3f88210a9d41'
        DOCKERHUB_REPO = 'aarthi1609/demo-project'
        SSH_CREDENTIALS_ID = '1f69c97f-6a8b-4718-b16c-b0061662773d'
        SERVER_IP = '43.205.238.84'
        PROJECT_NAME = 'demo-app'
        DOCKER_IMAGE_TAG = 'latest'
        }
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout code from GitHub
                checkout scm: [$class: 'GitSCM', branches: [[name: '*/main']], 
                                userRemoteConfigs: [[credentialsId: "${env.GIT_CREDENTIALS_ID}", 
                                                     url: 'https://github.com/aarthi1609/demo-project.git']]]
            }
        }

        // Step 2
        stage('Build by Maven') {
                steps {
                    sh 'mvn clean package'
                }
        }

        stage('SonarQube Analysis') {
            environment {
                // Set SonarQube environment variables
                scannerHome = tool 'SonarQube Scanner'
            }
            steps {
                // Run SonarQube analysis
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${PROJECT_NAME} -Dsonar.sources=src -Dsonar.java.binaries=target"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build Docker image
                script {
                    docker.build("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                // Login to Docker Hub and push the image
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDENTIALS}") {
                        docker.image("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                // Deploy the Docker image to the server using SSH
                script {
                    sshagent (credentials: ["${SSH_CREDENTIALS_ID}"]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no user@${SERVER_IP} << EOF
                        docker pull ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}
                        docker stop ${PROJECT_NAME} || true
                        docker rm ${PROJECT_NAME} || true
                        docker run -d --name ${PROJECT_NAME} -p 1222:8080 ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}
                        EOF
                        """
                    }
                }
            }
        }
    }

}      
