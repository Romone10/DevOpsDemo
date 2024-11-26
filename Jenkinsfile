pipeline {
    agent any 
    stages {
        stage('Checkout') { 
            steps {
                sh 'echo checkout'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'ba84ca1f-cbe4-48f7-83f7-d99f79c7f000', url: 'https://github.com/devopszhaw/DevOps-03-DevOpsDemo']])
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
            }
        }
        stage('Deploy') { 
            steps {
                sh 'echo deploy'
            }
        }
    }
}

