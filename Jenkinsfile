pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'develop', url: 'https://github.com/Empire843/affiliate-supporter'
            }
        }

        stage('Build & Test - Link Service') {
            steps {
                dir('link-service') {
                    sh 'mvn clean test'
                }
            }
        }

        stage('Build & Test - Crawler Service') {
            steps {
                dir('crawler-service') {
                    sh 'mvn clean test'
                }
            }
        }

        stage('Build & Test - Alert Service') {
            steps {
                dir('alert-service') {
                    sh 'mvn clean test'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker compose build'
            }
        }
    }

    post {
        failure {
            echo 'Build failed!'
        }
        success {
            echo 'Build passed!'
        }
    }
}