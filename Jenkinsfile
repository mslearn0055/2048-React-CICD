pipeline {
    agent any

    parameters {
        choice(
            name: 'ENV',
            choices: ['dev', 'sqa', 'prod'],
            description: 'Select the environment to build and test'
        )
        choice(
            name: 'ACTION',
            choices: ['build', 'test-xray', 'test-snyk', 'deploy'],
            description: 'Select what action to perform'
        )
    }

    stages {
        stage('Build') {
            when { expression { params.ACTION == 'build' } }
            steps {
                echo "🔨 Building for ${params.ENV}..."
                sh "npm install"
                sh "npm run build"
            }
        }

        stage('Xray Scan') {
            when { expression { params.ACTION == 'test-xray' } }
            steps {
                echo "🔎 Running Xray scan in ${params.ENV}..."
                sh "echo Running Xray Scan for ${params.ENV}"
            }
        }

        stage('Snyk Test') {
            when { expression { params.ACTION == 'test-snyk' } }
            steps {
                echo "🔐 Running Snyk test in ${params.ENV}..."
                sh "snyk test || true"
            }
        }

        stage('Deploy to Prod') {
            when { expression { params.ENV == 'prod' && params.ACTION == 'deploy' } }
            steps {
                echo "🚀 Deploying to Production..."
                sh "echo Deploying PROD build"
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline finished with ENV=${params.ENV}, ACTION=${params.ACTION}"
        }
    }
}
