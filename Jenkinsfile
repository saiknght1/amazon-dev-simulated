pipeline {
    agent any

    triggers {
        // Nightly trigger (~2 AM)
        cron('H 2 * * *')
    }

    stages {

        stage('Detect Branch & Configure') {
            steps {
                script {

                    echo "Branch: ${env.BRANCH_NAME}"

                    // Detect if triggered by cron
                    def isTimer = currentBuild.getBuildCauses('hudson.triggers.TimerTrigger$TimerTriggerCause')

                    // 🔥 Restrict cron runs ONLY to alpha branch
                    if (isTimer && env.BRANCH_NAME != 'alpha') {
                        echo "Skipping nightly run for non-alpha branch: ${env.BRANCH_NAME}"
                        currentBuild.result = 'NOT_BUILT'
                        return
                    }

                    // Branch-based logic
                    if (env.BRANCH_NAME == 'alpha') {
                        env.ENV = 'qa'
                        env.SUITE = isTimer ? 'testng-regression.xml' : 'testng-smoke.xml'

                    } else if (env.BRANCH_NAME == 'main') {
                        env.ENV = 'prod'
                        env.SUITE = 'testng-regression.xml'

                    } else {
                        // feature branches
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
                echo "Cloning Automation Repo..."
                git branch: 'main',
                    url: 'https://github.com/amazon-ui-automation-org/amazon-ui-automation.git'
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running ${env.SUITE} on ${env.ENV}"

                bat "mvn clean test -DsuiteXmlFile=suites/${env.SUITE} -Denv=${env.ENV}"
            }

            stage('Generate Allure Report') {
                steps {
                    echo "Generating Allure Report..."

                    allure includeProperties: false,
                           jdk: '',
                           results: [[path: 'target/allure-results']]
                }
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
        always {
            echo "Build completed for branch: ${env.BRANCH_NAME}"
        }
    }
}