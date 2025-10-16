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
                sh 'mvn clean'
            }
        }
        
        stage('Artifact Construction') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t gestion-station-skii:latest .'
            }
        }
        
        stage('Deploy Docker Image') {
            steps {
                sh '''
                    docker stop ski-app || true
                    docker rm ski-app || true
                    docker run -d --name ski-app -p 8080:8080 gestion-station-skii:latest
                '''
            }
        }
        
        stage('Update Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
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
