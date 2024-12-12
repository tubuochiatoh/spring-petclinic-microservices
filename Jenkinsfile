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
                    sh '''
                    if [ -d microservices ]; then
                        for service in $(ls microservices); do
                            echo "Building image for $service"
                            docker build -t ferdinandtubuo/${service}:latest ./microservices/${service}
                        done
                    else
                        echo "No microservices directory found!"
                    fi
                    '''
                }
            }
        }

        stage('Push Images to Dockerhub Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
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
            echo 'Cleaning up...'
            sh 'docker system prune -f'
            cleanWs()
        }
        success {
            junit '**/target/surefire-reports/*.xml'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
