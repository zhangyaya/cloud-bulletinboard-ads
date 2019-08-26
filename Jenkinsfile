@Library(['piper-lib', 'piper-lib-os']) _

/*
Pipeline structure copied from Piper's reference template: https://github.wdf.sap.corp/ContinuousDelivery/piper-library/blob/master/vars/sapPiperPipeline.groovy
Removed all stages which are not neeeded.
*/

pipeline {
    agent none
    options {
        skipDefaultCheckout()
        timestamps()
    }
    stages {
        stage('Init') {
            steps {
                library 'piper-lib-os'
                sapPiperStageInit script: this, customDefaults: params.customDefaults
            }
        }
        stage('Central Build') {
            // when {expression { env.BRANCH_NAME ==~ this.globalPipelineEnvironment.getStepConfiguration('', '').productiveBranch }}
            steps {
                sapPiperStageCentralBuild script: this
            }
        }
        stage('Deploy to Dev') {
            when {expression { env.BRANCH_NAME ==~ this.globalPipelineEnvironment.getStepConfiguration('', '').productiveBranch }}
            steps {
                sapPiperStageAcceptance script: this
            }
        }
    }
    post {
        /* https://jenkins.io/doc/book/pipeline/syntax/#post */
        success {setBuildStatus(currentBuild)}
        aborted {setBuildStatus(currentBuild, 'ABORTED')}
        failure {setBuildStatus(currentBuild, 'FAILURE')}
        unstable {setBuildStatus(currentBuild, 'UNSTABLE')}
        cleanup {
            sapPiperPublishNotifications script: this
            influxWriteData script: this, wrapInNode: true
            script {
                if(env.BRANCH_NAME ==~ this.globalPipelineEnvironment.getStepConfiguration('', '').productiveBranch){
                    if(this.globalPipelineEnvironment.configuration.runStep?.postAction?.slackSendNotification)
                        slackSendNotification script: this, wrapInNode: true
                }
            }
            mailSendNotification script: this, wrapInNode: true
        }
    }
}