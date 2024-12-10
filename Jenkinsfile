pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'       // Use Java 17
        maven 'Maven3'     // Use Maven 3
    }

    environment {
        APP_NAME = "register-app-pipeline"                  // Application name
        RELEASE = "1.0.0"                                   // Release version
        IMAGE_NAME = "yash407/${APP_NAME}"                  // Docker image name
        IMAGE_TAG = "${RELEASE}-${env.BUILD_NUMBER}"        // Docker image tag
        JENKINS_API_TOKEN = credentials('jenkins-api-token') // API token for triggering CD pipeline
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
                    withDockerRegistry([credentialsId: 'docker-hub-creds']) {
                        def docker_image = docker.build "${IMAGE_NAME}:${IMAGE_TAG}" // Build the Docker image
                        docker_image.push() // Push the image with the versioned tag
                        docker_image.push('latest') // Push the image with the 'latest' tag
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                script {
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table"
                }
            }
        }

        stage("Cleanup Artifacts") {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh """
                        curl -v -k --user clouduser:${JENKINS_API_TOKEN} \
                        -X POST -H 'cache-control: no-cache' \
                        -H 'content-type: application/x-www-form-urlencoded' \
                        --data 'IMAGE_TAG=${IMAGE_TAG}' \
                        'http://ec2-54-90-147-22.compute-1.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
            emailext body: '''${SCRIPT, template="groovy-html.template"}''',
                     subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful",
                     mimeType: 'text/html',
                     to: "yashdavkhar@gmail.com"
        }

        failure {
            echo "Pipeline failed. Check logs for details."
            emailext body: '''${SCRIPT, template="groovy-html.template"}''',
                     subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed",
                     mimeType: 'text/html',
                     to: "yashdavkhar@gmail.com"
        }
    }
}
