pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Build in Dev') {
            steps {
                echo "🔨 Building in DEV environment..."
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test in Dev') {
            when { succeeded() }
            parallel {
                stage('Xray Scan - Dev') {
                    steps {
                        echo "🔎 Running Xray Scan in Dev..."
                        // Replace with actual Xray scan command
                        sh 'echo "Xray Scan (Dev)"'
                    }
                }
                stage('Snyk Test - Dev') {
                    steps {
                        echo "🔐 Running Snyk Test in Dev..."
                        // Replace with actual snyk test
                        sh 'snyk test || true'
                    }
                }
            }
        }

        stage('Build in SQA') {
            steps {
                echo "🔨 Building in SQA environment..."
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test in SQA') {
            when { succeeded() }
            parallel {
                stage('Xray Scan - SQA') {
                    steps {
                        echo "🔎 Running Xray Scan in SQA..."
                        sh 'echo "Xray Scan (SQA)"'
                    }
                }
                stage('Snyk Test - SQA') {
                    steps {
                        echo "🔐 Running Snyk Test in SQA..."
                        sh 'snyk test || true'
                    }
                }
            }
        }

        stage('Build in Prod') {
            steps {
                echo "🔨 Building in PROD environment..."
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test in Prod') {
            when { succeeded() }
            parallel {
                stage('Xray Scan - Prod') {
                    steps {
                        echo "🔎 Running Xray Scan in Prod..."
                        sh 'echo "Xray Scan (Prod)"'
                    }
                }
                stage('Snyk Test - Prod') {
                    steps {
                        echo "🔐 Running Snyk Test in Prod..."
                        sh 'snyk test || true'
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            when { succeeded() }
            steps {
                echo "🚀 Deploying application to PROD..."
                // Replace with actual deployment commands
                sh 'echo "Deploying to Production Server..."'
            }
        }
    }

    post {
        always {
            echo "📋 Pipeline finished (success/failure)."
        }
    }
}
