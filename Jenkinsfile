pipeline {
    agent any

    triggers {
        cron('''
        H 2 * * *
        H 3 */2 * *
    ''')
    }

    environment {
        ENV = 'qa'
        SUITE = 'testng-smoke.xml'
        IS_PR = 'false'
    }

    stages {

        stage('Detect Branch & Configure') {
            steps {
                script {

                    echo "Branch: ${env.BRANCH_NAME}"

                    def isTimer = currentBuild.getBuildCauses('hudson.triggers.TimerTrigger$TimerTriggerCause')
                    def isPR = env.CHANGE_ID != null

                    env.IS_PR = isPR.toString()

                    // 🔥 PR BUILDS
                    if (isPR) {

                        def target = env.CHANGE_TARGET

                        if (target == 'alpha') {
                            env.ENV = 'qa'
                            env.SUITE = 'testng-smoke.xml'
                            echo "PR → feature to alpha → Smoke on QA (no deploy)"
                        }
                        else if (target == 'main') {
                            env.ENV = 'prod'
                            env.SUITE = 'testng-smoke.xml'
                            echo "PR → alpha to main → Smoke on PROD (no deploy)"
                        }
                    }

                    // 🟢 FEATURE BRANCH → SKIP DIRECT BUILDS
                    else if (env.BRANCH_NAME.startsWith('feature/')) {
                        echo "Skipping feature branch build (only PR builds allowed)"
                        currentBuild.result = 'ABORTED'
                        return
                    }

                    // 🔵 ALPHA BRANCH
                    else if (env.BRANCH_NAME == 'alpha') {

                        env.ENV = 'qa'
                        env.SUITE = isTimer ? 'testng-regression.xml' : 'testng-smoke.xml'

                        echo "Alpha Build → ${env.SUITE}"
                    }

                    // 🔴 MAIN BRANCH
                    else if (env.BRANCH_NAME == 'main') {

                        if (isTimer) {
                            env.ENV = 'prod'
                            env.SUITE = 'testng-smoke.xml'
                            echo "Scheduled PROD smoke run"
                        } else {
                            env.ENV = 'prod'
                            env.SUITE = 'testng-regression.xml'
                            echo "Main Build → Regression on PROD"
                        }
                    }

                    // ⏱️ Skip cron for non-alpha/main branches
                    if (isTimer && !(env.BRANCH_NAME in ['alpha', 'main'])) {
                        echo "Skipping scheduled run for branch: ${env.BRANCH_NAME}"
                        currentBuild.result = 'ABORTED'
                        return
                    }

                    echo "Final → ENV: ${env.ENV}, SUITE: ${env.SUITE}, IS_PR: ${env.IS_PR}"
                }
            }
        }

        // ✅ Deploy only for NON-PR builds
        stage('Deploy Application (Simulated)') {
            when {
                expression { env.IS_PR != 'true' }
            }
            steps {
                script {
                    echo "Deploying application to ${env.ENV}..."
                    sleep(time: 5, unit: 'SECONDS')
                    echo "Deployment completed for ${env.ENV}"
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
                echo "Running ${env.SUITE} on ${env.ENV}"

                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    bat "mvn clean test -DsuiteXmlFile=suites/${env.SUITE} -Denv=${env.ENV}"
                }
            }
        }

        stage('Generate Allure Report') {
            steps {
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
        aborted {
            echo "⚠️ Build intentionally skipped"
        }
        always {
            echo "Build completed for branch: ${env.BRANCH_NAME}"
        }
    }
}