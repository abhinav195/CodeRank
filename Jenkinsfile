pipeline {
    agent any
    tools {
        jdk 'jdk21'
        maven 'maven3'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
    }
}