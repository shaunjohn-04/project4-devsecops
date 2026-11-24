pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Starting Project 4 DevSecOps Pipeline'
                checkout scm
            }
        }

        stage('Verify Project') {
            steps {
                sh '''
                    echo "Checking project structure"
                    ls -la
                    ls -la project4/
                    ls -la project4/k8s/
                '''
            }
        }

        stage('Docker Check') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Pipeline Success') {
            steps {
                echo 'Jenkins pipeline is working successfully!'
            }
        }
    }
}
