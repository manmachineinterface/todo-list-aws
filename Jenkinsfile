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
                git branch: "${BRANCH}",
                    changelog: false,
                    poll: false,
                    url: "https://github.com/${REPOSITORY}"

                sh "curl -f -L https://raw.githubusercontent.com/manmachineinterface/todo-list-aws-config/refs/heads/${ENVIRONMENT}/samconfig.toml -o samconfig.toml"
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
                    withEnv(["ENV_NAME=${ENVIRONMENT}"]) {
                        sh '''
                            export BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-list-aws-$ENV_NAME --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text)
                            pytest --junitxml=result-rest.xml test/integration/todoApiTest.py::TestApi::test_api_listtodos \
                            test/integration/todoApiTest.py::TestApi::test_api_gettodo || exit
                        '''
                    }
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
