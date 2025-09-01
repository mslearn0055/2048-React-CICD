import org.json.JSONObject

def call() {
    pipeline {
        options {
            disableConcurrentBuilds()
            timestamps()
        }
        agent { label 'linux' }
        stages {
            stage('Initiate Pipeline') {
                steps {
                    ci_initiatePipeline()
                    checkout scm
                    ci_toolsDefinition()
                }
            }
            stage('Build Artifact') {
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        (params.'Operation' == 'Build')
                    }
                }
                steps {
                    ci_buildArtifact()
                    stash 'cicdStash'
                }
            }
            stage('Sonar Scan') {
                agent { label "${env.SONAR_AGENT}" }
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        (params.'Operation' == 'Build')
                    }
                }
                steps {
                    cleanWs()
                    unstash 'cicdStash'
                    ci_scanSonar()
                }
            }
            stage('Snyk Scan') {
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        (params.'Operation' == 'Build')
                    }
                }
                steps {
                    ci_scanSnyk()
                }
            }
            stage('XRay Scan') {
                when {
                    expression {
                        (params.'Operation' == 'Build')
                    }
                }
                steps {
                    ci_scanXRay()
                }
            }
            stage('Quality Gate') {
                when {
                    expression {
                        (params.'Operation' == 'Build')
                    }
                }
                steps {
                    ci_qualityGate()
                }
            }
            stage('Artifact Upload') {
                when {
                    expression {
                        (params.'Operation' == 'Build')
                    }
                }
                steps {
                    ci_uploadArtifact()
                }
            }
            stage('DEV Deployment') {
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        ((params.'Operation' == 'Build') || (params.'Operation' == 'Deploy') && (params.'Environment' == 'DEV'))
                    }
                }
                steps {
                    cd_deployArtifact_bst('DEV', params.'Artifact Version')
                }
            }
            stage('DEV SmokeTest') {
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        ((params.'Operation' == 'Build') || (params.'Operation' == 'Deploy' && params.'Environment' == 'DEV') || (params.'Operation' == 'Test' && params.'Environment' == 'DEV' && params.'Test Type' == 'Smoke'))
                    }
                }
                steps {
                    ct_smoke_execute('DEV')
                }
            }
            stage('SQA Deployment') {
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        ((params.'Operation' == 'Build') || (params.'Operation' == 'Deploy') && (params.'Environment' == 'SQA'))
                    }
                }
                steps {
                    timeout(time: 5, unit: 'MINUTES') {
                        input(message: 'Proceed to SQA Deployment?', ok: 'Proceed')
                    }
                    cd_deployArtifact_bst('SQA', params.'Artifact Version')
                }
            }
            stage('SQA SmokeTest') {
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        ((params.'Operation' == 'Build') || (params.'Operation' == 'Deploy') && (params.'Environment' == 'SQA') || (params.'Operation' == 'Test') && (params.'Environment' == 'SQA') && params.'Test')
                    }
                }
                steps {
                    ct_smoke_execute('SQA')
                }
            }
            stage('SQA RegressionTest') {
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        ((params.'Operation' == 'Build') || (params.'Operation' == 'Deploy') && (params.'Environment' == 'SQA') || (params.'Operation' == 'Test') && (params.'Environment' == 'SQA') && params.'Test Type' == 'Regression')
                    }
                }
                steps {
                    ct_regression_execute('SQA')
                }
            }
            stage('ServiceNow ChangeRecord') {
                when {
                    expression {
                        ((params.'Operation' == 'Build') || (params.'Operation' == 'Deploy') && (params.'CR' == '*'))
                    }
                }
            }
            stage('STG Deployment') {
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        (params.'Operation' == 'Deploy' && params.'Environment' == 'STG') && (params.'CR' == '*')
                    }
                }
                steps {
                    cr_snow_validate(params.'CR')
                    cd_deployArtifact_bst('STG', params.'Artifact Version')
                }
                post {
                    success {
                        cr_snow_close(params.'CR', 'success')
                    }
                    failure {
                        cr_snow_close(params.'CR', 'failure')
                    }
                }
            }
            stage('PROD Deployment') {
                tools {
                    maven "${env.CICD_MAVEN_VERSION}"
                    nodeJs "${env.CICD_NODE_VERSION}"
                    jdk "${env.CICD_JDK_VERSION}"
                }
                when {
                    expression {
                        (params.'Operation' == 'Deploy' && params.'Environment' == 'PROD') && (params.'CR' == '*')
                    }
                }
                steps {
                    cr_snow_validate(params.'CR')
                    cd_deployArtifact_bst('PROD', params.'Artifact Version')
                }
                post {
                    success {
                        cr_snow_close(params.'CR', 'success')
                    }
                    failure {
                        cr_snow_close(params.'CR', 'failure')
                    }
                }
            }
        }
        post {
            always {
                cm_alerts_pipelineStatus()
            }
        }
    }
}
