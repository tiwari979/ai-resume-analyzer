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
                echo '========== Stage 4: Deploy All Services =========='
                sh 'docker compose down --remove-orphans || true'
                sh 'docker compose up -d'
            }
        }

        stage('Health Check') {
            steps {
                echo '========== Stage 5: Health Check =========='
                sh 'sleep 20'
                sh 'curl -f http://localhost/health/user   || exit 1'
                sh 'curl -f http://localhost/health/resume || exit 1'
                sh 'curl -f http://localhost/health/ai     || exit 1'
                echo 'All services are healthy!'
            }
        }
    }

    post {
        success {
            echo '✅ BUILD SUCCESS - All stages passed!'
        }
        failure {
            echo '❌ BUILD FAILED - Check logs above'
            sh 'docker compose logs --tail=50'
        }
        always {
            echo 'Pipeline execution completed.'
        }
    }
}