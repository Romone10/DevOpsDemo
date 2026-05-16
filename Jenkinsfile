pipeline {
    agent any

    environment {
        JAVA_HOME = "/opt/jdks/jdk-25"
        DOCKER_HOST = "tcp://host.docker.internal:2375"

        IMAGE_NAME = "gallomor/devopsdemo"
        CONTAINER_NAME = "gallomor-devopsdemo"
        HOST_PORT = "3001"
        CONTAINER_PORT = "8080"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out current repository'
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'chmod +x ./gradlew'
                    sh './gradlew test'
                }

                junit stdioRetention: '', testResults: '**/test-results/test/*.xml'
            }
        }

        stage('Build Frontend') {
            steps {
                nodejs('24.11.1') {
                    dir('frontend') {
                        sh 'npm install'
                        sh 'npm run lint:html'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f $CONTAINER_NAME || true

                    docker run -d \
                      --name $CONTAINER_NAME \
                      -p $HOST_PORT:$CONTAINER_PORT \
                      $IMAGE_NAME:latest
                '''
            }
        }

        stage('Check App') {
            steps {
                sh '''
                    sleep 5
                    docker ps
                    curl -f http://host.docker.internal:$HOST_PORT || true
                '''
            }
        }
    }
}