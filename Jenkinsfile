pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_PROJECT = 'fitness-track-cicd'
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        MONGO_URI = credentials('MONGO_URI')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                    cat > server/.env << EOF
                    NODE_ENV=production
                    PORT=4001
                    MONGO_URI=${MONGO_URI}
                    EOF
                '''
            }
        }

        stage('Build and Start Containers') {
            steps {
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} build'
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} up -d'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'docker ps | grep fitness-track-cicd'
                sh 'sleep 10'
                sh 'curl -s http://localhost:4001 || true'
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
            sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} down || true'
        }
        always {
            archiveArtifacts artifacts: 'docker-compose.log', allowEmptyArchive: true
        }
    }
} 