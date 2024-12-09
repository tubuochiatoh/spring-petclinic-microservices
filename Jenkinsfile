pipeline {
    agent any
    tools {
        maven 'maven38'
    }

    stages {
        
        stage('Credential Scanner for detecting Secrets') {
            steps {
                script {
                    def buildUrl = env.BUILD_URL
                    sh "gitleaks detect -v --no-git --source . --report-format json --report-path secrets.json || exit 0"
                }
            }
        }    
        stage('Build pet clinic') {
            steps {
                sh "mvn clean install"
            }
        }

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
        
        stage('Package Petclinic App') {
            steps {
                script {
                    // Package the application (For example, create a JAR or WAR file)
                    sh 'mvn package'
                }
            }
            post {
                success {
                    // Archive the package artifact and put in a folder
                    archiveArtifacts artifacts: 'spring-petclinic-*/target/*.jar', allowEmptyArchive: true
                }
            }

               stage('Containerize Microservices') {
            steps {
                script {
                    echo 'Building Docker images for microservices...'
                    // Assuming a Dockerfile is in each microservice directory
                    sh '''
                    for service in $(ls microservices); do
                        docker build -t Ferdinandtubuo/${service}:latest ./microservices/${service}
                    done
                    '''
                }
            }    

        stage('Push Images to Docker Registry') {
            steps {
                script {
                    echo 'Pushing Docker images to registry...'
                    // Login to the Docker registry
                    sh 'docker login -u $Ferdinandtubuo -p $C@madonisquinta19902014 https://app.docker.com/'
                    
                    // Push images
                    sh '''
                    for service in $(ls microservices); do
                        docker push myregistry/${service}:latest
                    done
                    '''
                }
            } 
        }    
    }
}

post {
        always {
            echo 'Cleaning up...'
            sh 'docker system prune -f'
        }
    }
}    
