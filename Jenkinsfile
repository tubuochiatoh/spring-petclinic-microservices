pipeline {
    agent any
    tools {
        maven 'maven38'
    }

    stages {
        stage('Credential Scanner for detecting Secrets') {
            steps {
                script {
                    sh "gitleaks detect -v --no-git --source . --report-format json --report-path secrets.json || echo 'Secrets detected. Review secrets.json.'"
                }
            }
        }

        stage('Build Pet Clinic') {
            steps {
                sh "mvn clean install"
            }
        }

        stage('Test Petclinic') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package Petclinic App') {
            steps {
                sh 'mvn package'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
                }
            }
        }

        stage('Containerize Petclinic Application') {
            steps {
                script {
                    echo 'Building the Docker image for the Spring Petclinic application...'

                    // Define artifact and port for the Docker build
                    def artifactName = "spring-petclinic"
                    def exposedPort = 8080

                    // Build Docker image
                    sh """
                    docker build \
                        --build-arg ARTIFACT_NAME=${artifactName} \
                        --build-arg EXPOSED_PORT=${exposedPort} \
                        -t ferdinandtubuo/${artifactName}:latest \
                        -f Dockerfile .
                    """
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    echo 'Pushing the Docker image to Docker Hub...'

                    // Docker login and push
                    withCredentials([usernamePassword(credentialsId: 'Dockerhub_id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                        docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        docker push ferdinandtubuo/spring-petclinic:latest
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs() // Clean workspace after pipeline execution
        }

        success {
            echo 'Pipeline completed successfully!'
            // Publish test results
            junit '**/target/surefire-reports/*.xml'
        }

        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}