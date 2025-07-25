pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/sney2591/CycleKart.git'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Bike-App-Store \
                        -Dsonar.projectKey=Bike-App-Store
                    '''
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t bike-app-store ."
                        sh "docker tag bike-app-store koolkaramel/bike-app-store:latest"
                        sh "docker push koolkaramel/bike-app-store:latest"
                    }
                }
            }
        }

        stage("Trivy Image Scan") {
            steps {
                sh "trivy image koolkaramel/bike-app-store:latest > trivy.txt"
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                docker stop bike-app-store || true
                docker rm bike-app-store || true
                docker run -d --name bike-app-store -p 8081:80 koolkaramel/bike-app-store:latest
                '''
            }
        }
    }
}