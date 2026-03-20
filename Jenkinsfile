pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    stages {

        stage('Build') {
            steps {
                bat 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat 'mvn sonar:sonar'
                }
            }
        }

        stage('Package') {
            steps {
                bat 'mvn package'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t customer-order-api .'
            }
        }

        stage('Run Container') {
            steps {
                bat 'docker rm -f customer-order-api || exit 0'
                bat 'docker run -d -p 8080:8080 --name customer-order-api customer-order-api'
            }
        }

    }
}