pipeline {
    agent any
    tools {
        maven '3.9.9'  // Name of your Maven installation 
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
                sh 'mvn clean package'
            }
        } 
       /* stage('SonarQube Analysis') {
            steps{
                withSonarQubeEnv(installationName: 'sonar-local') {
                  sh "mvn clean verify sonar:sonar -Dsonar.projectKey=vulnado -Dsonar.projectName='vulnado'"
                }
            }
        } */
        /*stage('Dependency-Check') {
           steps {
               dependencyCheck additionalArguments: '', odcInstallation: 'dep-check-auto'
               dependencyCheckPublisher pattern: ''
               archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-report.xml', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
               sh ' rm -rf dependency-check-report.xml*'
           }
        }*/
        stage('Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '--nvdApiKey ${NVDAPIKEY}', odcInstallation: 'dep-check-auto'
                dependencyCheckPublisher pattern: ''
                archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-report.xml', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                sh ' rm -rf dependency-check-report.xml*'
            }
        }
       stage('Generate SBOM') {
            steps {
                sh '''
                syft scan dir:. --output cyclonedx-json=sbom.json
                '''
                archiveArtifacts allowEmptyArchive: true, artifacts: 'sbom*', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                sh ' rm -rf sbom*'
            }
        }
        stage('Secrets Detection') {
            steps {
                sh '${DETECTSECRETS}/detect-secrets scan . > secrets.txt'
                archiveArtifacts allowEmptyArchive: true, artifacts: 'secrets.txt', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                sh ' rm -rf secrets.txt'

            }
        }
        stage('SCA-SNYK') {
            steps {
                snykSecurity(
                  snykInstallation: 'snyk',
                  snykTokenId: 'snykToken',
                  failOnIssues: false,
                )
            }
        }
        stage('Container Security') {
            steps {
                sh 'grype vulnerables/web-dvwa:latest > grype.txt'
                archiveArtifacts allowEmptyArchive: true, artifacts: 'grype.txt', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                sh ' rm -rf grype.txt'
            }
        }

    }
}
