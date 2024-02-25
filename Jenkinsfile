pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    
    stages {
        
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout') {
            steps {
                git 'https://github.com/NatashaGanorkar/DevSecOps.git'
            }
        }
        
        stage('TruffleHog Scan') {
            steps {
                
                script {
                    def trufflehogOutput = sh(
                        script: 'docker run --rm -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --repo https://github.com/NatashaGanorkar/DevSecOps.git',
                        returnStdout: true
                    ).trim()

                    // Save the output to a file
                    writeFile file: 'trufflehog_output.xml', text: trufflehogOutput
                }
                archiveArtifacts(artifacts: 'trufflehog_output.xml', fingerprint: true)
            }
        }
    
        
        
        stage('KICS Scan') {
            steps {
                script {
                    def outputDir = '/var/lib/jenkins/workspace/DevSecOps/kics_results'
                    sh '/home/ubuntu/kics/bin/kics scan -p . --ci --output-path ' + outputDir + ' --ignore-on-exit results --queries-path /home/ubuntu/kics/assets/queries'
                }
                archiveArtifacts(artifacts: 'kics_results/**', fingerprint: true)
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=DevSecOps \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=DevSecOps '''
    
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                script {
                   sh 'trivy image sonarqube:lts-community -f json -o trivy-report.json'
                }
                
                archiveArtifacts artifacts: 'trivy-report.json', allowEmptyArchive: true
            }
        }

    }
}
