 pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'hemanth123/spring-petclinic:latest'  // Docker image name
        SONAR_HOST_URL = 'http://localhost:9000'
    }

    stages {
        stage('Clone Repository') {
            steps {
                cleanWs()  // Clean workspace to ensure a fresh clone
                git url: 'https://github.com/aHemanth123/spring-petclinic.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonar-token')
            }
            steps {
                sh "./mvnw sonar:sonar -Dsonar.projectKey=spring-petclinic -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        echo "🔄 Removing old Docker image if exists..."
                        docker rmi -f $DOCKER_IMAGE || true  
                        
                        echo "🐳 Building new Docker image..."
                        docker build -t $DOCKER_IMAGE .

                        echo "🔑 Logging into Docker Hub..."
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                        echo "🚀 Pushing Docker image to Docker Hub..."
                        docker push $DOCKER_IMAGE
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                echo "🛑 Stopping existing container (if running)..."
                docker stop spring-petclinic || true  
                
                echo "🗑 Removing old container (if exists)..."
                docker rm spring-petclinic || true  

                echo "🚀 Deploying new container..."
                docker run --rm -d --name spring-petclinic -p 8080:8080 $DOCKER_IMAGE
                """
            }
        }
    }
}
