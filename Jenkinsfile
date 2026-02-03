pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        TOKEN = credentials('classic-access-token')
        REPOSITORY = 'manmachineinterface/todo-list-aws.git'
        ENVIRONMENT = 'production'
        BRANCH = 'master'
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: "$BRANCH",
                    changelog: false,
                    poll: false,
                    url: "https://github.com/${REPOSITORY}"
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --no-fail-on-empty-changeset --no-confirm-changeset --config-env ${ENVIRONMENT}
                '''
            }
        }
    }

    post {
        cleanup {
            cleanWs(deleteDirs: true, disableDeferredWipeout: true)
        }
    }
}
