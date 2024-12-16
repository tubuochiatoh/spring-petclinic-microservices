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

        stage('Containerize Microservices') {
            steps {
                script {
                    echo 'Building Docker images for microservices...'
                    // Assuming a Dockerfile is in each microservice directory
                    sh '''
                    for service in $(ls microservices); do
                        docker build \
                        --build-arg ARTIFACT_NAME=service \
                        --build-arg EXPOSED_PORT=8080
                        -t ferdinandtubuo/${service}:latest \
                        -f ./microservices/$service/Dockerfile \
                        ./microservices/$service
                    done
                    '''
                }
            }
        }   

        stage('Push Images to Dockerhub Registry') {
            steps {
                script {
                    echo 'Pushing Docker images to registry...'
                    // Login to the Docker registry
                   withCredentials([usernamePassword(credentialsId: 'Dockerhub_id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD https://app.docker.com/'  

                    // Push images
                    sh '''
                    for service in $(ls microservices); do
                        docker push ferdinandtubuo/$service:latest
                    done
                    '''
                }
            }
        }
    } 
}
    

    post {
        always {
            cleanWs()
        }
      


    
        success {
            // Archive and publish test results of the spring-petclinic
            junit '**/target/surefire-reports/*.xml'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
