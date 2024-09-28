pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    environment {
        SNYK_TOKEN = credentials('snyk-api-token') 
    }
    stages {
        
        stage('Install Dependencies') { 
            steps {
                sh 'npm install'
                sh 'npm install -g snyk'
                echo 'Dependencies installed successfully!'
               
            }
        }
        stage('Snyk Security Scan') {
            steps {
                sh 'snyk auth $SNYK_TOKEN' 
                sh 'snyk test --severity-threshold=high' 
                echo 'Snyk security scan completed successfully!' 
            }
        }
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm run build' 
                echo 'Build completed successfully!' 
            }
        }
        
    }
    post {
        failure {
            echo 'Build failed due to critical vulnerabilities or other errors. Halting pipeline.' // Message on failure
        }
        success {
            echo 'Pipeline completed successfully!' // Message on success
        }
    }
}

