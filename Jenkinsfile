pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        PATH = "${PATH}:/opt/homebrew/bin"
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jaiswaladi246/fullstack-bank.git'
            }
        }
        
        stage('Install Trivy') {
            steps {
                withEnv(["PATH+HOME=/usr/local/bin:/opt/homebrew/bin/trivy"]) {
                    sh 'brew install trivy'
                }
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                withEnv(["PATH+TRIVY=/usr/local/bin"]) {
                    sh "trivy fs ."
                }
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./app/backend --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('SONARQUBE ANALYSIS') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=Bank -Dsonar.projectKey=Bank"
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('Backend') {
            steps {
                dir('app/backend') {
                    sh "npm install"
                }
            }
        }
        
        stage('Frontend') {
            steps {
                dir('app/frontend') {
                    sh "npm install"
                }
            }
        }
        
        stage('Deploy to Container') {
            steps {
                sh "npm run compose:up -d"
            }
        }
    }
}
