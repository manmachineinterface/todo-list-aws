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
        BRANCH = 'develop'
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: "${BRANCH}",
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

        stage('Rest Test') {
            steps {
                catchError(buildResult: 'ABORTED', stageResult: 'FAILURE') {
                    sh '''
                        export BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-list-aws-production --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text)
                        pytest --junitxml=result-rest.xml test/integration/todoApiTest.py || exit
                    '''
                    junit testResults: 'result-rest.xml', allowEmptyResults: false, skipPublishingChecks: true
                }
            }
        }

        stage('Promote') {
            steps {
                sh '''
                    git remote set-url origin https://${TOKEN}@github.com/${REPOSITORY}
                    git config merge.ours.driver true

                    git checkout master
                    git merge develop --no-commit --no-ff
                    git checkout HEAD -- Jenkinsfile
                    git commit -m "feat: merge develop into master preserving Jenkinsfile"
                    git push origin master

                    # Credentials and Identity configuration
                    git remote set-url origin https://${TOKEN}@github.com/${REPOSITORY}
                    git config user.email "jenkins@todo-list-aws.com"
                    git config user.name "Jenkins CI"

                    # Set merge driver to "keep"
                    git config merge.keep.driver true

                    # Create a dummy commit
                    git checkout develop
                    git pull origin develop
                    date > last_build.txt
                    git add last_build.txt .gitattributes
                    git commit -m "chore: update build timestamp [skip ci]" || echo "No changes"
                    git push origin develop

                    # Merge to master
                    git checkout master
                    git pull origin master
                    git merge develop -m "feat: merge develop into master (protected by gitattributes)"
                    git push origin master

                    # Back-merge
                    git checkout develop
                    git merge master -m "chore: sync branches [skip ci]"
                    git push origin develop
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
