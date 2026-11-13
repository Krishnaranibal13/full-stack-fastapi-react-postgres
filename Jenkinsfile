pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DEPLOY_DIR = '/home/ubuntu/full-stack-fastapi-react-postgres'
        APP_URL = 'http://127.0.0.1'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Files') {
            steps {
                sh '''
                    set -e

                    test -f docker-compose.yml
                    test -f backend/Dockerfile
                    test -f frontend/Dockerfile
                    test -f frontend/nginx.conf
                    test -f "$DEPLOY_DIR/.env"

                    echo "Project files and deployment environment verified."
                '''
            }
        }

        stage('Copy Latest Code') {
            steps {
                sh '''
                    set -e

                    rsync -a --delete \
                      --exclude='.git/' \
                      --exclude='.env' \
                      --exclude='__pycache__/' \
                      --exclude='*.pyc' \
                      "$WORKSPACE/" "$DEPLOY_DIR/"

                    echo "Latest source code copied."
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                dir('/home/ubuntu/full-stack-fastapi-react-postgres') {
                    sh '''
                        set -e
                        docker compose build backend frontend
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                dir('/home/ubuntu/full-stack-fastapi-react-postgres') {
                    sh '''
                        set -e
                        docker compose up -d --remove-orphans
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -e

                    echo "Waiting for application..."
                    sleep 15

                    docker compose -f "$DEPLOY_DIR/docker-compose.yml" ps

                    curl -fsS "$APP_URL/" > /dev/null
                    curl -fsS "$APP_URL/api/docs" > /dev/null

                    echo "Application deployment successful."
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Application deployed successfully!'
        }

        failure {
            echo 'FAILED: Deployment failed. Checking container logs.'

            sh '''
                docker compose -f "$DEPLOY_DIR/docker-compose.yml" \
                  logs --tail=50 || true
            '''
        }
    }
}

