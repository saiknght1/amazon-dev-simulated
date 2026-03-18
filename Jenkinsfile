pipeline {

agent any

stages {

    stage('Dev Triggered') {
        steps {
            echo "Dev repo triggered CI pipeline"
            echo "Branch: ${env.BRANCH_NAME}"
        }
    }

    stage('Run Automation from Automation Repo') {
        steps {
            echo "Cloning Automation Repo......"

            git branch: 'main',
                url: 'https://github.com/amazon-ui-automation-org/amazon-ui-automation.git'

            echo "Running Automation Tests..."

            bat "mvn clean test -DsuiteXmlFile=suites/testng-smoke.xml -Denv=qa"
        }
    }
}

}
