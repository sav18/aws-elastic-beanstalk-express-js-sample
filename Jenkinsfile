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
        
        stage('Snyk Security Scan') {
            steps {
                script {
                    echo 'Starting Snyk Security Scan...'

                    def snykResults = sh(script: './node_modules/.bin/snyk test --json', returnStdout: true)
                    def jsonResults = readJSON(text: snykResults)
                    
                    writeFile file: 'snyk-report.json', text: snykResults
                    
                    if (jsonResults.vulnerabilities.any { it.severity == 'critical' }) {
                        error("Critical vulnerabilities found! Failing the build.")
                    } else {
                        echo 'No critical vulnerabilities found. Proceeding...'
                    }
                }
            }
            post {
                success {
                    echo 'Snyk Security Scan completed successfully!'
                }
                failure {
                    echo 'Snyk Security Scan failed due to critical vulnerabilities.'
                }
            }
        }

        stage('Test Phase') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
                echo 'Tests completed successfully.'
            }
            post {
                success {
                    echo 'All tests passed!'
                }
                failure {
                    echo 'Some tests failed. Please check the logs for details.'
                }
            }
        }
    }
    
    post {
        always {
            script {
                node {
                    echo 'Archiving Snyk vulnerability scan report...'
                    archiveArtifacts artifacts: 'snyk-report.json', allowEmptyArchive: true
                }
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
