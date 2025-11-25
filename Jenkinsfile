pipeline {
    agent any
    
    environment {
        WORKSPACE = "${WORKSPACE}"
        COMPOSE_FILE = "${WORKSPACE}/docker-compose-jenkins.yml"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning repository from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/wardakhan0101/WebTechSemProject-Jenkins.git'
            }
        }
        
        stage('Stop Old Containers') {
            steps {
                echo 'Stopping any existing Jenkins containers...'
                sh '''
                    cd ${WORKSPACE}
                    docker-compose -f docker-compose-jenkins.yml down || true
                '''
            }
        }
        
        stage('Deploy Containers') {
            steps {
                echo 'Starting containers with docker-compose...'
                sh '''
                    cd ${WORKSPACE}
                    docker-compose -f docker-compose-jenkins.yml up -d
                '''
            }
        }
        
        stage('Wait for Services') {
            steps {
                echo 'Waiting for services to be ready...'
                sh 'sleep 20'
            }
        }
        
        stage('Import Database') {
            steps {
                echo 'Importing database data...'
                sh '''
                    docker exec jenkins-bookverse-mongodb mongoimport \
                        --db bookverse \
                        --collection books \
                        --file /tmp/db_files/BOOKVERSE.books.json \
                        --jsonArray \
                        --drop || true
                    
                    docker exec jenkins-bookverse-mongodb mongoimport \
                        --db bookverse \
                        --collection genres \
                        --file /tmp/db_files/BOOKVERSE.genres.json \
                        --jsonArray \
                        --drop || true
                    
                    docker exec jenkins-bookverse-mongodb mongoimport \
                        --db bookverse \
                        --collection users \
                        --file /tmp/db_files/BOOKVERSE.users.json \
                        --jsonArray \
                        --drop || true
                    
                    docker exec jenkins-bookverse-mongodb mongoimport \
                        --db bookverse \
                        --collection wishlists \
                        --file /tmp/db_files/BOOKVERSE.wishlists.json \
                        --jsonArray \
                        --drop || true
                '''
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying containers are running...'
                sh '''
                    docker-compose -f ${WORKSPACE}/docker-compose-jenkins.yml ps
                    echo "=========================================="
                    echo "✅ Application accessible at:"
                    echo "   Frontend: http://3.110.179.129:3001"
                    echo "   Backend:  http://3.110.179.129:5001"
                    echo "=========================================="
                '''
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
            echo 'Jenkins automated build and deployment finished.'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
            sh 'docker-compose -f ${WORKSPACE}/docker-compose-jenkins.yml logs || true'
        }
    }
}
