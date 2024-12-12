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
                        docker build -t ferdinandtubuo/${service}:latest ./microservices/${service}
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
                    sh 'docker login -u $ferdinandtubuo -p $C@madonisquinta19902014 https://app.docker.com/'

                    // Push images
                    sh '''
                    for service in $(ls microservices); do
                        docker push ferdinandtubuo/${service}:latest
                    done
                    '''
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
