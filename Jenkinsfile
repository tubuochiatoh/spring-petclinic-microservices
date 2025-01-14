pipeline {
    agent any
    tools {
        maven 'maven38'
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

//         stage('Install Snyk CLI') {
//             steps {
//                sh '''
//                curl -fsSL https://github.com/snyk/snyk/releases/latest/download/snyk-linux -o snyk
//                chmod +x snyk
//                sudo mv snyk /usr/local/bin
//                snyk --version
//                '''
//     }
// }


        //   stage('Snyk Dependency Scan') {
        //       environment {
        //         SNYK_TOKEN = credentials('Snyk_API_Token') // use credentials securely
        //       }
        //     steps {
        //         script {
        //             // Run Snyk CLI to scan for vulnerabilities in dependencies
        //             sh """
        //             snyk auth $SNYK_TOKEN
        //             snyk test --all-projects
        //             """
        //         }
        //     }

        //      post {
        //         failure {
        //             echo "Snyk found vulnerabilities in project dependencies!"
        //         }
        //     }
        // }

        //  stage('Snyk Scan') {
            
        //     steps {
        //     echo 'Testing...'
        //     snykSecurity(
        //         snykInstallation: 'snyk@latest',
        //         snykTokenId: 'snyk-jenkins-token',
        //         // place other parameters here
        //     )
        //     }
        // }
       
         stage('Test Petclinic') {
            steps {
                script {
                    //Run Unit Test
                    sh 'mvn test'
                }
            }
            post {
                always {
                    //Archive and publish test results of the spring-petclinic"
                    junit '**/target/surefire-reports/*.xml'
                }
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
                    for (service in MICROSERVICES) {
                        echo "Building Docker image for ${service}"
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

         stage('Install Snyk CLI') {
            steps {
               sh '''
               curl -fsSL https://github.com/snyk/snyk/releases/latest/download/snyk-linux -o snyk
               chmod +x snyk
               sudo mv snyk /usr/local/bin
               snyk --version
               '''
    }
}

        stage('Snyk Scan') {
                    
                    steps {
                    echo 'Testing...'
                    snykSecurity(
                        snykInstallation: 'snyk@latest',
                        snykTokenId: 'snyk-jenkins-token',
                        // place other parameters here
                    )
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
                            "spring-petclinic-config-server",
                            "spring-petclinic-customers-service",
                            "spring-petclinic-discovery-server",
                            "spring-petclinic-vets-service",
                            "spring-petclinic-visits-service"
                        ]
                        for (service in MICROSERVICES) {
                            echo "Pushing Docker image for ${service}"
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

    }
}
