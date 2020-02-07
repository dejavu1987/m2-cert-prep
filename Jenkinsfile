pipeline {
    agent { label "my127ws" }
    environment {
        MY127WS_KEY = credentials('magento2-sample-my127ws-key')
        MY127WS_ENV = "pipeline"
    }
    triggers { cron(env.BRANCH_NAME == 'develop' ? 'H H(0-6) * * *' : '') }
    stages {
        stage('Build') {
            steps { sh 'ws install' }
        }
        stage('Test')  {
            parallel {
                stage('quality')    { steps { sh 'ws exec composer test-quality'    } }
                stage('unit')       { steps { sh 'ws exec composer test-unit'       } }
                stage('acceptance') { steps { sh 'ws exec composer test-acceptance' } }
            }
        }
        stage('Publish') {
            when { not { triggeredBy 'TimerTrigger' } }
            steps { sh 'ws app publish' }
        }
        stage('Deploy (QA)') {
            environment {
                DO_ACCESS_TOKEN = credentials('inviqa-digitalocean-token')
            }
            when {
                not { triggeredBy 'TimerTrigger' }
                branch 'develop'
            }
            steps {
                sh 'ws app deploy qa'
            }
        }
    }
    post {
        always {
            sh 'ws destroy'
            cleanWs()
        }
    }
}
