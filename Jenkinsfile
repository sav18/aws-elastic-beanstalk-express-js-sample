pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage(' Install Dependencies ') {
            steps {
                sh 'npm install --save'
                echo 'Installing  Complete'

                sh 'npm install snyk --save-dev'
                echo 'Snyk Installation completed'

                withCredentials([string(credentialsId: 'snyk_token', variable: 'SNYK_TOKEN')]) {
                    sh './node_modules/.bin/snyk auth $SNYK_TOKEN'
                    echo 'Snyk Authentication Completed'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'npm install --save'
                echo 'Build Phase Completed'
            }
        }

        stage('Snyk Security Scan') {
            steps {
                 script {
                    def snykPoints = sh(script: './node_modules/.bin/snyk test --json', returnStdout: true)
                    def jsonPoints = readJSON(text: snykPoints)
                    if (jsonPoints.vulnerabilities.any { it.severity == 'critical' }) {
                        error("Vulnerabilities found")
                    } else {
                        writeFile file: 'snyk-report.json', text: snykResults
                    }
                }

                echo ' Security Scan Completed'
            }
            post {
                success {
                    echo ' Security Scan passed!'
                }
                failure {
                    echo 'Failed.'
                }
            }
        }


        stage('Test Phase') {
            steps {
                script {
                    sh 'npm test'
                    echo 'Tests completed.'
                }
            }
            post {
                success {
                    echo 'Tests passed!'
                }
                failure {
                    echo 'Some tests failed.'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline Completed.'
        }
    }
}
