pipeline {
    agent any

    environment {
        COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '========== Stage 1: Checkout Code =========='
                checkout scm
            }
        }

        stage('Install & Test') {
            parallel {

                stage('Test user-service') {
                    steps {
                        dir('user-service') {
                            sh 'npm install'
                            sh 'npm test --if-present'
                        }
                    }
                }

                stage('Test resume-service') {
                    steps {
                        dir('resume-service') {
                            sh 'npm install'
                            sh 'npm test --if-present'
                        }
                    }
                }

                stage('Test ai-service') {
                    steps {
                        dir('ai-service') {
                            sh 'npm install'
                            sh 'npm test --if-present'
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo '========== Stage 3: Build Docker Images =========='
                sh 'docker compose build --no-cache'
            }
        }

        stage('Deploy Services') {
            steps {
                echo '========== Stage 4: Deploy Services =========='

                sh '''
                docker compose up -d --build \
                    nginx \
                    user-service \
                    resume-service \
                    ai-service \
                    frontend
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo '========== Stage 5: Health Check =========='

                sh 'sleep 30'

                sh 'curl -f http://user-service:3001/health'
                sh 'curl -f http://resume-service:3002/health'
                sh 'curl -f http://ai-service:3003/health'

                echo 'All services are healthy!'
            }
        }
    }

    post {

        success {
            echo 'BUILD SUCCESS - All stages passed!'
        }

        failure {
            echo 'BUILD FAILED - Check logs above'

            sh '''
            docker compose ps
            docker compose logs --tail=100
            '''
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}