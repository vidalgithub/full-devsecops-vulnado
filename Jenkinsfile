pipeline {
    agent any
    tools {
        maven '3.9.9'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn --version'
                sh 'mvn clean package'
            }
        } /*
        stage('SonarQube Analysis') {
            steps{
                withSonarQubeEnv(installationName: 'sonar-local') {
                  sh "mvn clean verify sonar:sonar -Dsonar.projectKey=vulnado -Dsonar.projectName='vulnado'"
                }
            }
        }*/
    }
}
