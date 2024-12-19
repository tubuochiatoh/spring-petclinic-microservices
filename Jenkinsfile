pipeline {
    agent any
    tools {
        maven 'maven38'
    }

    environment {
        ARTIFACT_NAME = 'spring-petclinic' // Replace with your artifact name
        EXPOSED_PORT = '8080' // Replace with your port
    }

    stages {
        stage('Credential Scanner for detecting Secrets') {
            steps {
                sh """
                gitleaks detect -v --no-git --source . \
                --report-format json --report-path secrets.json || \
                echo 'Secrets detected. Review secrets.json.'
                """
            }
        }

        stage('Build Pet Clinic') {
            steps {
                sh "mvn clean install"
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
                    def MICROSERVICES = [
                        "spring-petclinic-admin-server",
                        "spring-petclinic-api-gateway",
                        "spring-petclinic-config-server",
                        "spring-petclinic-customers-service",
                        "spring-petclinic-discovery-server",
                        "spring-petclinic-vets-service",
                        "spring-petclinic-visits-service"
                    ]
                    MICROSERVICES.each { service ->
                        sh """
                        docker build \
                            --build-arg ARTIFACT_NAME=${service} \
                            --build-arg EXPOSED_PORT=8080 \
                            -t ferdinandtubuo/${service}:3.2.7 \
                            .
                        """
                    }
                }
            }
        }

        stage('Push Images to Dockerhub Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Dockerhub_id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD https://index.docker.io/v1/'
                        def MICROSERVICES = [
                            "spring-petclinic-admin-server",
                            "spring-petclinic-api-gateway",
                            "spring-petclinic-config-server"
                            "spring-petclinic-customers-service",
                            "spring-petclinic-discovery-server",
                            "spring-petclinic-vets-service",
                            "spring-petclinic-visits-service"
                        ]
                        MICROSERVICES.each { service ->
                            sh "docker push ferdinandtubuo/${service}:3.2.7"
                        }
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
