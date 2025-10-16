pipeline {
    agent any
    
    stages {
        stage('Checkout Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Siwartaktak/Project_DevOps_Siwar.git'
            }
        }
        
        stage('Maven Clean') {
            steps {
                bat 'mvn clean'
            }
        }
        
        stage('Artifact Construction') {
            steps {
                bat 'mvn package'
            }
        }
        
        stage('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                bat 'mvn deploy'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                bat 'docker build -t gestion-station-skii:latest .'
            }
        }
        
        stage('Deploy Docker Image') {
            steps {
                bat '''
                    docker stop ski-app 2>nul || echo Container not running
                    docker rm ski-app 2>nul || echo Container not found
                    docker run -d --name ski-app -p 8080:8080 gestion-station-skii:latest
                '''
            }
        }
        
        stage('Update Kubernetes') {
            steps {
                bat 'kubectl apply -f k8s/'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Pipeline finished!'
        }
    }
}
