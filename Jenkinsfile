pipeline {
    agent any
    tools {
       /* maven '3.9.9'  // Name of your Maven installation */
        dependencyCheck 'dep-check-auto' // Name of your Dependency-Check installation
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        } /*
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        } */
       /* stage('SonarQube Analysis') {
            steps{
                withSonarQubeEnv(installationName: 'sonar-local') {
                  sh "mvn clean verify sonar:sonar -Dsonar.projectKey=vulnado -Dsonar.projectName='vulnado'"
                }
            }
        } */
        stage('Dependency-Check') {
           steps {
               dependencyCheck additionalArguments: '', odcInstallation: 'dep-check-auto'
               dependencyCheckPublisher pattern: ''
               archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-report.xml', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
               sh ' rm -rf dependency-check-report.xml*'
           }
        }
    }
}
