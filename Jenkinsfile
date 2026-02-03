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

        stage('Rest Test') {
            steps {
                catchError(buildResult: 'ABORTED', stageResult: 'FAILURE') {
                    sh '''
                        export BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-list-aws-production --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text)
                        pytest --junitxml=result-rest.xml -k "test_api_get test_api_list" test/integration/todoApiTest.py || exit
                    '''
                    junit testResults: 'result-rest.xml', allowEmptyResults: false, skipPublishingChecks: true
                }
            }
        }
    }

    post {
        cleanup {
            cleanWs(deleteDirs: true, disableDeferredWipeout: true)
        }
    }
}
