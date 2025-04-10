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
                    withCredentials([usernamePassword(credentialsId: 'Docker_hub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
	                    echo "Logging in to Docker Hub..."
	                    docker login -u $DOCKER_USER -p $DOCKER_PASS

	                    echo "Building Docker image..."
	                    docker build -t $DOCKER_USER/java-microservice-cicd:latest .

	                    echo "Pushing Docker image..."
	                    docker push $DOCKER_USER/java-microservice-cicd:latest
	                """
	 	    }
	        }
	    }
	}

        stage('Kubernetes Deploy') {
            steps {
                script {
                    sh '''
                        echo "Deleting existing deployment (if exists)..."
                        kubectl delete -f kubernetes/deployment.yaml --ignore-not-found
                        echo "Applying deployments and service yaml"
                        kubectl apply -f kubernetes/deployment.yaml
                        kubectl apply -f kubernetes/service.yaml
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

