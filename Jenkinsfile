pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'selenium-python-tests:latest'
        FRONTEND_URL = 'http://13.235.74.237:5173'
        BACKEND_URL = 'http://13.235.74.237:5000'
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
        
/*      stage('Publish Test Results') {
            steps {
                echo 'Publishing test results...'
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'selenium_tests',
                    reportFiles: 'report.html',
                    reportName: 'Selenium Test Report'
                ])
            }
        } 
*/
    }
    
    post {
        always {
            echo 'Cleaning up workspace...'
        }
        
        success {
            echo 'Tests passed successfully! Sending email...'
            emailext(
                subject: "✅ Jenkins Build Success: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                    <h2>Build Success!</h2>
                    <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                    <p><strong>Status:</strong> SUCCESS</p>
                    <p><strong>Duration:</strong> ${currentBuild.durationString}</p>
                    <p><strong>Commit:</strong> ${env.GIT_COMMIT}</p>
                    <p><strong>Branch:</strong> ${env.GIT_BRANCH}</p>
                    <br>
                    <p>All Selenium tests passed successfully!</p>
                    <p><a href="${env.BUILD_URL}Selenium_20Test_20Report/">View Test Report</a></p>
                    <p><a href="${env.BUILD_URL}console">View Console Output</a></p>
                """,
                to: "${env.GIT_COMMITTER_EMAIL}",
                mimeType: 'text/html',
                attachLog: false
            )
        }
        
        failure {
            echo 'Tests failed! Sending email...'
            emailext(
                subject: "❌ Jenkins Build Failed: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                    <h2>Build Failed!</h2>
                    <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                    <p><strong>Status:</strong> FAILURE</p>
                    <p><strong>Duration:</strong> ${currentBuild.durationString}</p>
                    <p><strong>Commit:</strong> ${env.GIT_COMMIT}</p>
                    <p><strong>Branch:</strong> ${env.GIT_BRANCH}</p>
                    <br>
                    <p>Some tests failed. Please check the logs.</p>
                    <p><a href="${env.BUILD_URL}Selenium_20Test_20Report/">View Test Report</a></p>
                    <p><a href="${env.BUILD_URL}console">View Console Output</a></p>
                """,
                to: "${env.GIT_COMMITTER_EMAIL}",
                mimeType: 'text/html',
                attachLog: true
            )
        }
    }
}
