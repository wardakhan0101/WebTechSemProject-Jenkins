pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'selenium-python-tests:latest'
        FRONTEND_URL = 'http://172.31.8.94:5173'
        BACKEND_URL = 'http://172.31.8.94:5000'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling code from GitHub...'
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                echo 'Verifying environment...'
                sh '''
                    echo "Frontend URL: $FRONTEND_URL"
                    echo "Backend URL: $BACKEND_URL"
                    echo "Docker Image: $DOCKER_IMAGE"
                    docker images | grep selenium-python-tests || echo "Warning: Docker image not found"
                '''
            }
        }

        stage('Run Selenium Tests') {
            steps {
                echo 'Running Selenium tests in Docker container...'
                script {
                    try {
                        sh '''
                            docker run --rm \
                                --network host \
                                -v ${WORKSPACE}/selenium_tests:/tests \
                                -e FRONTEND_URL=${FRONTEND_URL} \
                                -e BACKEND_URL=${BACKEND_URL} \
                                ${DOCKER_IMAGE} \
                                pytest /tests/test_bookverse.py -v --html=/tests/report.html --self-contained-html
                        '''
                        currentBuild.result = 'SUCCESS'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Tests failed: ${e.message}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
        }

        success {
            script {
                def committerEmail = sh(
                    script: "git log -1 --pretty=format:'%ae'",
                    returnStdout: true
                ).trim()
                
                echo "Tests passed successfully! Sending email to ${committerEmail}..."
                emailext(
                    subject: "✅ Jenkins Build Success: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                    body: """
                        <h2>Build Success!</h2>
                        <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                        <p><strong>Status:</strong> SUCCESS ✅</p>
                        <p><strong>Duration:</strong> ${currentBuild.durationString}</p>
                        <p><strong>Commit:</strong> ${env.GIT_COMMIT}</p>
                        <p><strong>Committer:</strong> ${committerEmail}</p>
                        <br>
                        <p><strong>All 15 Selenium tests passed successfully!</strong></p>
                        <p><a href="${env.BUILD_URL}console">View Console Output</a></p>
                    """,
                    to: "${committerEmail}",
                    mimeType: 'text/html',
                    attachLog: false
                )
            }
        }

        failure {
            script {
                def committerEmail = sh(
                    script: "git log -1 --pretty=format:'%ae'",
                    returnStdout: true
                ).trim()
                
                echo "Tests failed! Sending email to ${committerEmail}..."
                emailext(
                    subject: "❌ Jenkins Build Failed: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                    body: """
                        <h2>Build Failed!</h2>
                        <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                        <p><strong>Status:</strong> FAILURE ❌</p>
                        <p><strong>Duration:</strong> ${currentBuild.durationString}</p>
                        <p><strong>Commit:</strong> ${env.GIT_COMMIT}</p>
                        <p><strong>Committer:</strong> ${committerEmail}</p>
                        <br>
                        <p><strong>Some tests failed. Please check the logs.</strong></p>
                        <p><a href="${env.BUILD_URL}console">View Console Output</a></p>
                    """,
                    to: "${committerEmail}",
                    mimeType: 'text/html',
                    attachLog: true
                )
            }
        }
    }
}
