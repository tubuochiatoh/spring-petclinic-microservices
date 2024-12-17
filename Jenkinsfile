pipeline {
    agent any
    tools {
        maven 'maven38'
    }

    stages {
        stage('Credential Scanner for detecting Secrets') {
            steps {
                script {
                    sh """
                    gitleaks detect -v --no-git --source . \
                    --report-format json --report-path secrets.json || \
                    echo 'Secrets detected. Review secrets.json.'
                    """
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
                sh "mvn test"
            }
        }

        stage('Package Petclinic App') {
            steps {
                sh "mvn package"
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
                    def MICROSERVICE = "spring-petclinic-admin-server spring-petclinic-api-gateway spring-petclinic-config-server spring-petclinic-customers-service spring-petclinic-discovery-server spring-petclinic-vets-service spring-petclinic-visits-service"

                    sh '''
                    for ARTIFACT_NAME in ''' + MICROSERVICE + '''; do
                        docker build -t ferdinandtubuo/${ARTIFACT_NAME}:3.2.7 .
                    done
                    '''
                }
            }
        }

        stage('Push Images to Dockerhub Registry') {
            steps {
                script {
                    echo 'Pushing Docker images to registry...'
                    withCredentials([usernamePassword(credentialsId: 'Dockerhub_id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD https://index.docker.io/v1/'

                        def MICROSERVICE = "spring-petclinic-admin-server spring-petclinic-api-gateway spring-petclinic-config-server spring-petclinic-customers-service spring-petclinic-discovery-server spring-petclinic-vets-service spring-petclinic-visits-service"

                        sh '''
                        for ARTIFACT_NAME in ''' + MICROSERVICE + '''; do
                            docker push ferdinandtubuo/${ARTIFACT_NAME}:3.2.7
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
            junit '**/target/surefire-reports/*.xml'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}