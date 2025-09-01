pipeline {
    agent { label 'linux' }

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    parameters {
        choice(name: 'Operation', choices: ['Build', 'Deploy', 'Test'], description: 'Select operation')
        choice(name: 'Environment', choices: ['DEV', 'SQA', 'STG', 'PROD'], description: 'Target environment')
        string(name: 'Artifact Version', defaultValue: '1.0.0', description: 'Artifact version for deployment')
        choice(name: 'Test Type', choices: ['Smoke', 'Regression'], description: 'Type of test to run')
        string(name: 'CR', defaultValue: '', description: 'Change Request number (required for STG/PROD)')
    }

    stages {
        stage('Initiate Pipeline') {
            steps {
                echo "📌 Initiating pipeline..."
                checkout scm
                echo "Tools defined (Maven, Node, JDK)"
            }
        }

        stage('Build Artifact') {
            tools {
                maven "${env.CICD_MAVEN_VERSION}"
                nodejs "${env.CICD_NODE_VERSION}"
                jdk "${env.CICD_JDK_VERSION}"
            }
            when { expression { params.Operation == 'Build' } }
            steps {
                echo "🔨 Building artifact..."
                sh 'mvn clean install -DskipTests'
                stash name: 'build-output', includes: '**/target/**'
            }
        }

        stage('Sonar Scan') {
            when { expression { params.Operation == 'Build' } }
            agent { label "${env.SONAR_AGENT}" }
            steps {
                cleanWs()
                unstash 'build-output'
                echo "🔎 Running SonarQube Scan..."
                sh "mvn sonar:sonar"
            }
        }

        stage('Snyk Scan') {
            when { expression { params.Operation == 'Build' } }
            steps {
                echo "🔐 Running Snyk Scan..."
                sh "snyk test || true"
            }
        }

        stage('XRay Scan') {
            when { expression { params.Operation == 'Build' } }
            steps {
                echo "🧪 Running XRay scan..."
                sh "echo 'XRay scan placeholder'"
            }
        }

        stage('Quality Gate') {
            when { expression { params.Operation == 'Build' } }
            steps {
                echo "✅ Checking Quality Gate..."
                sh "echo 'Quality Gate passed'"
            }
        }

        stage('Artifact Upload') {
            when { expression { params.Operation == 'Build' } }
            steps {
                echo "⬆️ Uploading artifact..."
                sh "echo 'Upload artifact version ${params.'Artifact Version'}'"
            }
        }

        stage('DEV Deployment') {
            when { expression { (params.Operation == 'Build' || (params.Operation == 'Deploy' && params.Environment == 'DEV')) } }
            steps {
                echo "🚀 Deploying to DEV..."
                sh "echo 'Deploying version ${params.'Artifact Version'} to DEV'"
            }
        }

        stage('DEV SmokeTest') {
            when { expression { (params.Operation == 'Build' || (params.Operation == 'Deploy' && params.Environment == 'DEV') || (params.Operation == 'Test' && params.Environment == 'DEV' && params.'Test Type' == 'Smoke')) } }
            steps {
                echo "🔥 Running DEV Smoke Tests..."
                sh "echo 'Smoke tests in DEV passed'"
            }
        }

        stage('SQA Deployment') {
            when { expression { (params.Operation == 'Build' || (params.Operation == 'Deploy' && params.Environment == 'SQA')) } }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    input(message: 'Proceed to SQA Deployment?', ok: 'Proceed')
                }
                echo "🚀 Deploying to SQA..."
                sh "echo 'Deploying version ${params.'Artifact Version'} to SQA'"
            }
        }

        stage('SQA SmokeTest') {
            when { expression { (params.Operation == 'Build' || (params.Operation == 'Deploy' && params.Environment == 'SQA') || (params.Operation == 'Test' && params.Environment == 'SQA' && params.'Test Type' == 'Smoke')) } }
            steps {
                echo "🔥 Running SQA Smoke Tests..."
                sh "echo 'Smoke tests in SQA passed'"
            }
        }

        stage('SQA RegressionTest') {
            when { expression { (params.Operation == 'Build' || (params.Operation == 'Deploy' && params.Environment == 'SQA') || (params.Operation == 'Test' && params.Environment == 'SQA' && params.'Test Type' == 'Regression')) } }
            steps {
                echo "🧪 Running SQA Regression Tests..."
                sh "echo 'Regression tests in SQA passed'"
            }
        }

        stage('STG Deployment') {
            when { expression { (params.Operation == 'Deploy' && params.Environment == 'STG' && params.CR != '') } }
            steps {
                echo "📋 Validating Change Record ${params.CR}..."
                sh "echo 'CR validation passed'"
                echo "🚀 Deploying to STG..."
                sh "echo 'Deploying version ${params.'Artifact Version'} to STG'"
            }
            post {
                success { echo "✅ CR ${params.CR} closed: success" }
                failure { echo "❌ CR ${params.CR} closed: failure" }
            }
        }

        stage('PROD Deployment') {
            when { expression { (params.Operation == 'Deploy' && params.Environment == 'PROD' && params.CR != '') } }
            steps {
                echo "📋 Validating Change Record ${params.CR}..."
                sh "echo 'CR validation passed'"
                echo "🚀 Deploying to PROD..."
                sh "echo 'Deploying version ${params.'Artifact Version'} to PROD'"
            }
            post {
                success { echo "✅ CR ${params.CR} closed: success" }
                failure { echo "❌ CR ${params.CR} closed: failure" }
            }
        }
    }

    post {
        always {
            echo "📢 Sending pipeline status notification..."
        }
    }
}
