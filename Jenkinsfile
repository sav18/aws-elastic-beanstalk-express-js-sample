pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    environment {
        SNYK_TOKEN = credentials('6ec59578-9b80-4981-af86-f0c4336d2b34') 
    }
    stages {
        stage('Install Dependencies') { 
            steps {
                sh 'npm install'
                sh 'npm install -g snyk'
                echo 'Dependencies installed successfully!'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm run build' 
                echo 'Build completed successfully!' 
            }
        }
        
        stage('Snyk Security Scan Phase') {
            steps {
                script {
                    def snykResults = sh(script: './node_modules/.bin/snyk test --json', returnStdout: true)
                    def jsonResults = readJSON(text: snykResults)
                    writeFile file: 'snyk-report.json', text: snykResults
                    if (jsonResults.vulnerabilities.any { it.severity == 'critical' }) {
                        error("Critical vulnerabilities found! Failing the build.")
                    }
                }
                echo 'Security Scan Completed'
            }
        }

        stage('Test Phase') {
            steps {
                script {
                    sh 'npm test'
                    echo 'Tests completed.'
                }
            }
        }
    }
    post {
        always {
            node {
                script {
                    archiveArtifacts artifacts: 'snyk-report.json', allowEmptyArchive: true
                    echo 'Pipeline completed. Artifacts archived.'
                }
            }
        }
        failure {
            echo 'Pipeline failed. Check logs and reports for details.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}
