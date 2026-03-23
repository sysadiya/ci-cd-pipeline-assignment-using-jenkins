pipeline {
    agent any

    stages {

        stage('Clone Repository') {
            steps {
                git 'https://github.com/sysadiya/ci-cd-pipeline-assignment-using-jenkins.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Code Analysis') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage('Package Application') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t customer-order-api .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker run -d -p 8082:8080 customer-order-api'
            }
        }
    }
}