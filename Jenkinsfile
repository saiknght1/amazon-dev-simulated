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

                    def isTimer = currentBuild.getBuildCauses('hudson.triggers.TimerTrigger$TimerTriggerCause')

                    // Restrict cron to alpha only
                    if (isTimer && env.BRANCH_NAME != 'alpha') {
                        echo "Skipping nightly run for non-alpha branch: ${env.BRANCH_NAME}"
                        currentBuild.result = 'NOT_BUILT'
                        return
                    }

                    if (env.BRANCH_NAME == 'alpha') {
                        env.ENV = 'qa'
                        env.SUITE = isTimer ? 'testng-regression.xml' : 'testng-smoke.xml'

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
                echo "Cloning Automation Repo..."
                git branch: 'main',
                    url: 'https://github.com/amazon-ui-automation-org/amazon-ui-automation.git'
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running ${env.SUITE} on ${env.ENV}"

                // ✅ Prevent pipeline from stopping on failure
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    bat "mvn clean test -DsuiteXmlFile=suites/${env.SUITE} -Denv=${env.ENV}"
                }
            }
        }

        // ✅ SEPARATE STAGE (VERY IMPORTANT)
        stage('Generate Allure Report') {
            steps {
                echo "Generating Allure Report..."

                allure includeProperties: false,
                       jdk: '',
                       results: [[path: 'target/allure-results']]
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