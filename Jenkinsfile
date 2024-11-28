pipeline {
    agent any
    options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    tools {
        maven '3.9.9'  // Name of your Maven installation 
    }
    environment {
        TARGET_URL = 'http://testphp.vulnweb.com/'
        ZAP_PATH = '/var/lib/jenkins/ZAP_2.15.0/zap.sh'
        ZAP_API_KEY = '90auo26dsn27jujq9pbopsai8v' // Get this from ZAP UI if necessary
        ZAP_PORT = '8081'
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
        } /*
        stage('SonarQube Analysis') {
            steps{
                withSonarQubeEnv(installationName: 'sonarqube-10.7') {
                 sh '''
                    mvn clean verify sonar:sonar \
                      -Dsonar.projectKey=demoapp-project \
                      -Dsonar.projectName='demoapp-project' \
                      -Dsonar.host.url=http://sonarqube.beitcloud.com:9000 \
                      -Dsonar.token=sqp_b1fe7cd624cca76539df32df978e4c4e0433f6b3
                 '''
                    //sh "mvn clean package sonar:sonar"
                    //sh "mvn clean verify sonar:sonar -Dsonar.projectKey=vulnado -Dsonar.projectName='vulnado'"
                }
            }
        } */
        /*
        stage('Dependency Check - ODC') {
            steps {
                dependencyCheck additionalArguments: '--nvdApiKey ${NVDAPIKEY}', odcInstallation: 'dep-check-auto'
                dependencyCheckPublisher pattern: ''
                archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-report.xml', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                sh ' rm -rf dependency-check-report.xml*'
            }
        }*/
        /*stage('Dependency Check - ODC') { 
            steps {
                // Run dependency check
                dependencyCheck additionalArguments: '--nvdApiKey ${NVDAPIKEY}', odcInstallation: 'dep-check-auto'
                
                // Publish the report
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                
                // Archive the report
                archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-report.xml', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                
                // Remove the report files after the build
                //#sh 'rm -rf dependency-check-report.xml*'
                
                // Check for severity levels (HIGH, CRITICAL, MEDIUM) and fail the job if found
                script {
                    // Use cat and grep to check for the patterns in the report
                    def severityCheck = sh(script: "cat dependency-check-report.xml | grep -i -E 'critical|high|medium'", returnStatus: true)
                    
                    // If the grep command finds any matching lines, it will return a non-zero status, so we fail the build
                    if (severityCheck != 0) {
                        error("Build failed due to critical, high, or medium severity issues in dependency check report")
                    }
                }
            }
        }*/
        stage('Dependency Check - ODC') {  
            steps {
                // Run dependency check
                dependencyCheck additionalArguments: '--nvdApiKey ${NVDAPIKEY}', odcInstallation: 'dep-check-auto'
                
                // Publish the report
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                
                // Archive the report
                archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-report.xml', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                
                // Remove the report files after the build
                sh 'rm -rf dependency-check-report.xml*'
                
                // Check for 'confidence="LOW"' and fail the job if found
                script {
                    // Use cat and grep to check for 'confidence="LOW"' in the report
                    def lowConfidenceCheck = sh(script: "cat dependency-check-report.xml | grep -i 'confidence=\"LOW\"'", returnStatus: true)
                    
                    // If the grep command finds any matching lines, it will return a non-zero status, so we fail the build
                    if (lowConfidenceCheck == 0) {
                        error("Build failed due to low confidence issues in the dependency check report")
                    }
                }
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
        stage('Secrets Detection - DETECT-SECRETS') {
            steps {
                sh '${DETECTSECRETS}/detect-secrets scan . > secrets.txt'
                archiveArtifacts allowEmptyArchive: true, artifacts: 'secrets.txt', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                sh ' rm -rf secrets.txt'

            }
        }
        stage('SCA - SNYK') {
            steps {
                snykSecurity(
                  snykInstallation: 'snyk',
                  snykTokenId: 'snykToken',
                  failOnIssues: false,
                )
            }
        }
        stage('Container Security - GRYPE') {
            steps {
                sh 'grype vulnerables/web-dvwa:latest > grype.txt'
                archiveArtifacts allowEmptyArchive: true, artifacts: 'grype.txt', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                sh ' rm -rf grype.txt'
            }
        }/*
        stage('DAST - OWASP ZAP') {
            steps {
                
              sh'''
                zap-cli start --start-options -daemon
                zap-cli status
                zap-cli open-url ${TARGET_URL}
                zap-cli spider ${TARGET_URL}
                zap-cli active-scan ${TARGET_URL}
                zap-cli -v report -o report-zap-cli-jenkins.md -f md
                zap-cli shutdown
                '''  
                archiveArtifacts allowEmptyArchive: true, artifacts: 'report-zap-cli-jenkins.md', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                sh ' rm -rf report-zap-cli-jenkins.md'
            }
        }*/
    }
}
