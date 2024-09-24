pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
                echo 'Install Dependencies Finished'

                sh 'npm install snyk --save-dev'
                echo 'Snyk Installed'

                withCredentials([string(credentialsId: 'snyk_t', variable: 'SNYK_TOKEN')]) {
                    sh './node_modules/.bin/snyk auth $SNYK_TOKEN'
                    echo 'Snyk Authentication Finished'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'npm install --save'
                echo 'Install Dependencies Finished'
            }
        }

        stage('Snyk Security Scan') {
            steps {
                 script {
                    def snykResults = sh(script: './node_modules/.bin/snyk test --json', returnStdout: true)
                    def jsonResults = readJSON(text: snykResults)
                    if (jsonResults.vulnerabilities.any { it.severity == 'critical' }) {
                        error("Vulnerabilities found! Check snyk-report.json.")
                    } else {
                        writeFile file: 'snyk-report.json', text: snykResults
                    }
                }

                echo 'Snyk Security Scan Completed'
            }
            post {
                success {
                    echo 'Sync Security Scan passed!'
                }
                failure {
                    echo 'Failed.'
                }
            }
        }


        stage('Test') {
            steps {
                script {
                    sh 'npm install jest'
                    sh 'npm test'
                    echo 'Tests completed.'
                }
            }
            post {
                success {
                    echo 'Tests passed!'
                }
                failure {
                    echo 'Some tests failed. Check logs for details.'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
