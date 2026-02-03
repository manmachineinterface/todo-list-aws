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

        stage('Static test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh '''
                        flake8 --exit-zero --format=pylint src > flake8.out
                    '''

                    recordIssues tools: [flake8(name:'Flake8', pattern: 'flake8.out')]
                }

                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh '''
                        bandit -r ./src -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                    '''

                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
                }
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
