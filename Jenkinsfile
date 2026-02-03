pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        TOKEN = credentials('classic-access-token')
        REPOSITORY = 'manmachineinterface/todo-list-aws.git'
        ENVIRONMENT = 'staging'
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop',
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

        stage('Promote') {
            steps {
                sh '''
                    git switch master
                    git merge --strategy-option=ours develop
                    git push https://${TOKEN}@github.com/${REPOSITORY} master
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
