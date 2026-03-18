pipeline {
    agent any

    triggers {
        cron('H 2 * * *')
    }

    stages {

        stage('Detect Branch & Configure') {
            steps {
                script {
                    echo "Branch: ${env.BRANCH_NAME}"

                    if (env.BRANCH_NAME == 'alpha') {
                        env.ENV = 'qa'
                        env.SUITE = 'testng-smoke.xml'

                        def isTimer = currentBuild.getBuildCauses('hudson.triggers.TimerTrigger$TimerTriggerCause')

                        if (isTimer) {
                            echo "Nightly build detected"
                            env.SUITE = 'testng-regression.xml'
                        }

                    } else if (env.BRANCH_NAME == 'main') {
                        env.ENV = 'prod'
                        env.SUITE = 'testng-regression.xml'

                    } else {
                        env.ENV = 'qa'
                        env.SUITE = 'testng-smoke.xml'
                    }

                    echo "Environment: ${env.ENV}"
                    echo "Suite: ${env.SUITE}"
                }
            }
        }

        stage('Clone Automation Repo') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/amazon-ui-automation-org/amazon-ui-automation.git'
            }
        }

        stage('Run Tests') {
            steps {
                bat "mvn clean test -DsuiteXmlFile=suites/${env.SUITE} -Denv=${env.ENV}"
            }
        }
    }

    post {
        success {
            echo "✅ Tests Passed"
        }
        failure {
            echo "❌ Tests Failed"
        }
    }
}