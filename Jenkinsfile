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
                bat '''
                    docker stop test-mysql 2>nul || echo MySQL container not running
                    docker rm test-mysql 2>nul || echo MySQL container does not exist
                    docker run -d --name test-mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -e MYSQL_DATABASE=skidb -p 3307:3306 mysql:latest
                    powershell -Command "Start-Sleep -Seconds 40"
                    echo MySQL container started and ready
                '''
            }
        }

        stage('Maven Clean') {
            steps {
                bat 'mvn clean'
            }
        }

        stage('Compile') {
            steps {
                bat 'mvn compile'
            }
        }

        stage('Unit Tests') {
            steps {
                bat 'mvn test -Dspring.datasource.url=jdbc:mysql://localhost:3307/skidb?createDatabaseIfNotExist=true'
            }
        }

        stage('Artifact Construction') {
            steps {
                bat 'mvn package -DskipTests'
            }
        }

        stage('Publish to Nexus') {
            steps {
                bat 'mvn deploy -s settings.xml -DskipTests'
                
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
        always {
            bat 'docker stop test-mysql 2>nul || echo MySQL cleanup done'
            bat 'docker rm test-mysql 2>nul || echo MySQL cleanup done'
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

