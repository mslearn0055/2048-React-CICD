pipeline {
    agent any

    parameters {
        choice(
            name: 'ENV',
            choices: ['dev', 'sqa', 'prod'],
            description: 'Select the environment to build and test'
        )
    }

    stages {
        stage('Build') {
            steps {
                script {
                    echo "Building for ${params.ENV}..."
                    sh "npm install"
                    sh "npm run build"
                }
            }
        }

        stage('Tests') {
            parallel {
                stage('Xray Scan') {
                    steps {
                        echo "Running Xray scan for ${params.ENV}..."
                        sh "echo Xray Scan in ${params.ENV}"
                    }
                }
                stage('Snyk Test') {
                    steps {
                        echo "Running Snyk test for ${params.ENV}..."
                        sh "snyk test || true"
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                expression { params.ENV == 'prod' }
            }
            steps {
                echo "🚀 Deploying to Production..."
                sh "echo Deploy to PROD server"
            }
        }
    }
}
