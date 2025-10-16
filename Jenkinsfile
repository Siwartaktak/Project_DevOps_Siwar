pipeline {
    agent any
    stages {
        stage('Checkout Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Siwartaktak/Project_DevOps_Siwar.git'
            }
        }
        stage('Setup MySQL Container') {
            steps {
                sh '''
                    docker stop test-mysql 2>/dev/null || echo "MySQL container not running"
                    docker rm test-mysql 2>/dev/null || echo "MySQL container does not exist"
                    docker run -d --name test-mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -e MYSQL_DATABASE=skidb -p 3307:3306 mysql:latest
                    sleep 40
                    echo "MySQL container started and ready"
                '''
            }
        }
        stage('Maven Clean') {
            steps {
                sh 'mvn clean'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'mvn test -Dspring.datasource.url=jdbc:mysql://localhost:3307/skidb?createDatabaseIfNotExist=true'
            }
        }
        stage('Artifact Construction') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
        stage('Publish to Nexus') {
            steps {
                sh 'mvn deploy -s settings.xml -DskipTests'
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
                    docker stop ski-app 2>/dev/null || echo "Container not running"
                    docker rm ski-app 2>/dev/null || echo "Container not found"
                    docker run -d --name ski-app -p 8089:8080 gestion-station-skii:latest
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
        always {
            sh 'docker stop test-mysql 2>/dev/null || echo "MySQL cleanup done"'
            sh 'docker rm test-mysql 2>/dev/null || echo "MySQL cleanup done"'
            echo 'Pipeline finished!'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
