pipeline {
    agent { label 'Jenkins-Agent' }
    
    tools {
        jdk 'Java17'       // Use Java 17
        maven 'Maven3'     // Use Maven 3
    }
    
    environment {
        APP_NAME = "register-app-pipeline"                  // Application name
        RELEASE = "1.0.0"                                   // Release version
        DOCKER_USER = "yash407"  
        DOCKER_PASS = 'Y@shdavkhar1'                      // Docker Hub username
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"           // Docker image name
        IMAGE_TAG = "${RELEASE}-${env.BUILD_NUMBER}"        // Docker image tag
                             
    }
    
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs() // Cleanup workspace
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', 
                    credentialsId: 'github', 
                    url: 'https://github.com/yashdavkhar/register-app'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package" // Compile and package the application
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test" // Run unit tests
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar" // Perform SonarQube analysis
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false // Wait for quality gate status
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        def docker_image = docker.build "${IMAGE_NAME}:${IMAGE_TAG}" // Build the Docker image
                        docker_image.push() // Push the image with the versioned tag
                        docker_image.push('latest') // Push the image with the 'latest' tag
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully." // Log success message
        }
        failure {
            echo "Pipeline failed. Check logs for details." // Log failure message
        }
    }
}
