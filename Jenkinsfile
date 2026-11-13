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

                    echo "Project files verified successfully."
                '''
            }
        }

        stage('Copy Latest Code') {
            steps {
                sh '''
                    set -e

                    rsync -r --delete \
                      --omit-dir-times \
                      --no-perms \
                      --no-owner \
                      --no-group \
                      --exclude='.git/' \
                      --exclude='.env' \
                      --exclude='__pycache__/' \
                      --exclude='*.pyc' \
                      --exclude='node_modules/' \
                      --exclude='build/' \
                      --exclude='venv/' \
                      "$WORKSPACE/" "$DEPLOY_DIR/"

                    echo "Latest code copied successfully."
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                dir('/home/ubuntu/full-stack-fastapi-react-postgres') {
                    sh '''
                        set -e

                        docker compose build backend frontend

                        echo "Docker images built successfully."
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

                        echo "Application containers deployed."
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -e

                    echo "Waiting for application startup..."
                    sleep 15

                    docker compose -f "$DEPLOY_DIR/docker-compose.yml" ps

                    echo "Checking frontend..."
                    curl -fsS "$APP_URL/" > /dev/null

                    echo "Checking backend API..."
                    curl -fsS "$APP_URL/api/docs" > /dev/null

                    echo "Frontend and backend are responding successfully."
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Notes & Checklists application deployed!'
            echo 'Application URL: http://98.93.142.48'
        }

        failure {
            echo 'FAILED: Deployment pipeline encountered an error.'

            sh '''
                docker compose -f "$DEPLOY_DIR/docker-compose.yml" \
                  ps || true

                docker compose -f "$DEPLOY_DIR/docker-compose.yml" \
                  logs --tail=50 || true
            '''
        }

        always {
            echo 'Jenkins pipeline execution completed.'
        }
    }
}

