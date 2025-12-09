pipeline {
    agent any
    environment {
        JAVA_HOME = "/opt/jdks/jdk-25"
    }
    stages {
        stage('Checkout') { 
            steps {
                sh 'echo checkout'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'GitHub-mosazhaw', url: 'https://github.com/devopszhaw/DevOpsDemo']])
            }
        }
        stage('Build') { 
            steps {
                sh 'echo build'
                dir('backend') {
                    sh 'chmod +x ./gradlew'
                    sh './gradlew test'    
                }
                jacoco()
                junit stdioRetention: '', testResults: '**/test-results/test/*.xml'
                nodejs('NodeJS 22.11.0') {
                    dir('frontend') {
                        sh 'npm install'
                        sh 'npm run lint:html'                    
                    }
                }
                withCredentials([string(credentialsId: 'Sonarqube-Backend', variable: 'TOKEN')]) {
                    dir('backend') {
                        sh './gradlew sonar -Dsonar.projectKey=DevOpsDemo-Backend -Dsonar.projectName=\'DevOpsDemo-Backend\' -Dsonar.host.url=http://sonarqube:9000 -Dsonar.token=$TOKEN'    
                    }                    
                }
                withCredentials([string(credentialsId: 'Sonarqube-Frontend', variable: 'TOKEN')]) {
                    dir('frontend') {
                        nodejs('NodeJS 22.11.0') {
                            sh 'npx sonar-scanner -Dsonar.host.url=http://sonarqube:9000 -Dsonar.projectKey=DevOpsDemo-Frontend -Dsonar.projectName=\'DevOpsDemo-Frontend\' -Dsonar.token=$TOKEN'    
                        }
                    }                    
                }
            }
        }
        stage('Docker') {
            steps {
                sh '''
                    export DOCKER_HOST=tcp://host.docker.internal:2375
                    docker build -t mosazhaw/devopsdemo .
                '''
            }
        }
    }
}

