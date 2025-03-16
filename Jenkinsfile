pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'hemanth123/spring-petclinic:latest'  // Define your Docker image name
        SONAR_HOST_URL = 'http://localhost:9000'
    }

    stages {
        stage('Clone Repository') {
            steps {
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
                        docker rmi -f $DOCKER_IMAGE || true  # Remove any existing image to avoid conflicts
                        docker build -t $DOCKER_IMAGE .
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                docker stop spring-petclinic || true  # Stop running container if exists
                docker rm spring-petclinic || true  # Remove old container if exists
                docker run --rm -d --name spring-petclinic -p 8080:8080 $DOCKER_IMAGE
                """
            }
        }
    }
}
