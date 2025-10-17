pipeline {
    agent any

    environment {
        MYSQL_CONTAINER = 'test-mysql'
        MYSQL_DB = 'skidb'
        APP_CONTAINER = 'ski-app'
        APP_IMAGE = 'gestion-station-skii:latest'
        GIT_URL = 'https://github.com/Siwartaktak/Project_DevOps_Siwar.git'
        NETWORK_NAME = 'mynetwork'
    }

    stages {
        stage('Checkout Git') {
            steps {
                echo 'Cloning repository...'
                git branch: 'main', url: "${GIT_URL}"
            }
        }

        stage('Setup Docker Network') {
            steps {
                echo 'Creating Docker network if not exists...'
                sh '''
                    docker network inspect ${NETWORK_NAME} >/dev/null 2>&1 || \
                    docker network create ${NETWORK_NAME}
                '''
            }
        }

        stage('Setup MySQL Container') {
            steps {
                echo 'Setting up MySQL container...'
                sh '''
                    docker stop ${MYSQL_CONTAINER} 2>/dev/null || echo "MySQL container not running"
                    docker rm ${MYSQL_CONTAINER} 2>/dev/null || echo "MySQL container does not exist"

                    docker run -d --name ${MYSQL_CONTAINER} --network ${NETWORK_NAME} \
                        -e MYSQL_ALLOW_EMPTY_PASSWORD=yes \
                        -e MYSQL_DATABASE=${MYSQL_DB} \
                        mysql:latest

                    echo "Waiting for MySQL to be ready..."
                    MAX_ATTEMPTS=30
                    for i in $(seq 1 $MAX_ATTEMPTS); do
                        if docker exec ${MYSQL_CONTAINER} mysqladmin ping -h"127.0.0.1" --silent; then
                            echo "✅ MySQL is ready after $i attempts!"
                            break
                        fi
                        echo "⏳ Waiting for MySQL to start... ($i/$MAX_ATTEMPTS)"
                        sleep 5
                    done

                    if ! docker exec ${MYSQL_CONTAINER} mysqladmin ping -h"127.0.0.1" --silent; then
                        echo "❌ MySQL did not become ready in time."
                        docker logs ${MYSQL_CONTAINER}
                        exit 1
                    fi
                '''
            }
        }

        stage('Maven Clean') {
            steps {
                echo 'Running mvn clean...'
                sh 'mvn clean'
            }
        }

        stage('Compile') {
            steps {
                echo 'Compiling project...'
                sh 'mvn compile'
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'Running unit tests...'
                // Use the container hostname within the Docker network
                sh 'mvn test -Dspring.datasource.url=jdbc:mysql://test-mysql:3306/${MYSQL_DB}?createDatabaseIfNotExist=true'
            }
        }

        stage('Artifact Construction') {
            steps {
                echo 'Building artifact...'
                sh 'mvn package -DskipTests'
            }
        }

        stage('Publish to Nexus') {
            steps {
                echo 'Deploying artifact to Nexus...'
                sh 'mvn deploy -s settings.xml -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${APP_IMAGE} .'
            }
        }

        stage('Deploy Docker Image') {
            steps {
                echo 'Deploying Docker container...'
                sh '''
                    docker stop ${APP_CONTAINER} 2>/dev/null || echo "Container not running"
                    docker rm ${APP_CONTAINER} 2>/dev/null || echo "Container not found"

                    docker run -d --name ${APP_CONTAINER} --network ${NETWORK_NAME} \
                        -p 8089:8080 ${APP_IMAGE}
                '''
            }
        }

        stage('Update Kubernetes') {
            steps {
                echo 'Updating Kubernetes deployment...'
                sh 'kubectl apply -f k8s/'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up MySQL container...'
            sh '''
                docker stop ${MYSQL_CONTAINER} 2>/dev/null || echo "MySQL container already stopped"
                docker rm ${MYSQL_CONTAINER} 2>/dev/null || echo "MySQL container already removed"
            '''
            echo 'Pipeline finished!'
        }

        success {
            echo '✅ Pipeline completed successfully!'
        }

        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
