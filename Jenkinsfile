pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'MySonarQube' 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Wait for Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    sh '''
                        docker build -t prashanth7993/java-microservice-cicd:latest .
                        docker push prashanth7993/java-microservice-cicd:latest
                    '''
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                script {
                    sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "JAR built, analyzed by SonarQube, and archived successfully."
        }
        failure {
            echo "Build failed due to errors or Quality Gate failure."
        }
    }
}

