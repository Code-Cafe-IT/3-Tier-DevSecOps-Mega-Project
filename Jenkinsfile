pipeline {
    agent any
    tools{
        nodejs 'nodejs23.1.0'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Code-Cafe-IT/3-Tier-DevSecOps-Mega-Project.git'
            }
        }
        stage('Frontend Compilation') {
            steps {
                dir('client') {
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        
        stage('Backend Compilation') {
            steps {
                dir('api') {
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        
        stage('GitLeaks Scan') {
            steps {
                sh 'gitleaks detect --source ./client --exit-code 1 --report-path report.json'
                sh 'gitleaks detect --source ./api --exit-code 1 --report-path report.json'
                sh 'pwd'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=NodeJS-Project \
                            -Dsonar.projectKey=NodeJS-Project '''
                }
            }
        }
        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                   waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-jenkins'
                }
            }
        }
        stage('Trivy FS Scan'){
            steps{
                sh 'trivy fs . --format table -o fs-report.html'
            }
        }
        
    }
}
