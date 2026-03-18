pipeline {
    agent any

    environment {
        ENV = ''
        SUITE = ''
    }

    triggers {
        // Nightly regression (runs at 2 AM)
        cron('H 2 * * *')
    }

    stages {

        stage('Detect Branch & Configure') {
            steps {
                script {
                    echo "Branch: ${env.BRANCH_NAME}"

                    if (env.BRANCH_NAME == 'alpha') {
                        ENV = 'qa'
                        SUITE = 'testng-smoke.xml'

                        // If triggered by cron → run regression instead
                        if (currentBuild.rawBuild.getCause(hudson.triggers.TimerTrigger$TimerTriggerCause)) {
                            echo "Nightly build detected"
                            SUITE = 'testng-regression.xml'
                        }

                    } else if (env.BRANCH_NAME == 'main') {
                        ENV = 'prod'
                        SUITE = 'testng-regression.xml'

                    } else {
                        // feature branches
                        ENV = 'qa'
                        SUITE = 'testng-smoke.xml'
                    }

                    echo "Environment: ${ENV}"
                    echo "Suite: ${SUITE}"
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
                echo "Running ${SUITE} on ${ENV}"

                bat "mvn clean test -DsuiteXmlFile=suites/${SUITE} -Denv=${ENV}"
            }
        }
    }

    post {
        success {
            echo "✅ Tests Passed.."
        }
        failure {
            echo "❌ Tests Failed"
        }
    }
}