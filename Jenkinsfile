pipeline {
    agent any
    tools{
        nodejs 'nodejs23.1.0'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_USERNAME = 'minhduccloud'
        IMAGE_NAME_FRONTEND = 'frontend'
        IMAGE_NAME_BACKEND = 'backend'
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
                sh 'gitleaks detect --source ./client --exit-code 1 --report-path report-client.json'
                sh 'gitleaks detect --source ./api --exit-code 1 --report-path report-api.json'
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
        stage('Build-Tag & Push Backend Docker Image'){
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-token') {
                        dir('api') {
                            def dockerImage = docker.image("${DOCKERHUB_USERNAME}/${IMAGE_NAME_BACKEND}:${env.BUILD_NUMBER}")
                            echo "Pushing image ${dockerImage.id} to Docker Hub..."
                            dockerImage.push()
                            echo "Tagging image as 'latest' and pushing..."
                            dockerImage.tag('latest')
                            dockerImage.push('latest')
                            sh 'trivy image --format table -o frontend-image-report.html ${DOCKERHUB_USERNAME}/${IMAGE_NAME_BACKEND}:${env.BUILD_NUMBER}'
                        }
                    }
                }
            }
        }
        stage('Build-Tag & Push Frontend Docker Image'){
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-token') {
                        dir('api') {
                            def dockerImage = docker.image("${DOCKERHUB_USERNAME}/${IMAGE_NAME_FRONTEND}:${env.BUILD_NUMBER}")
                            echo "Pushing image ${dockerImage.id} to Docker Hub..."
                            dockerImage.push()
                            echo "Tagging image as 'latest' and pushing..."
                            dockerImage.tag('latest')
                            dockerImage.push('latest')
                            sh 'trivy image --format table -o frontend-image-report.html ${DOCKERHUB_USERNAME}/${IMAGE_NAME_FRONTEND}:${env.BUILD_NUMBER}'
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                echo 'Cleaning up local Docker image...'
                sh "docker rmi ${DOCKERHUB_USERNAME}/${IMAGE_NAME_BACKEND}:${env.BUILD_NUMBER} || true"
                sh "docker rmi ${DOCKERHUB_USERNAME}/${IMAGE_NAME_FRONTEND}:${env.BUILD_NUMBER} || true"
                sh "docker rmi ${DOCKERHUB_USERNAME}/${IMAGE_NAME_BACKEND}:latest || true"
                sh "docker rmi ${DOCKERHUB_USERNAME}/${IMAGE_NAME_FRONTEND}:latest || true"
            }
        }
    }
}
