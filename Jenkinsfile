pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_PROJECT = 'fitness-track-cicd'
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        MONGO_URI = credentials('MONGO_URI')
        JWT_SECRET = credentials('JWT_SECRET')
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
                    JWT_SECRET=${JWT_SECRET}
                    EOF
                '''
            }
        }

        stage('Build and Start Containers') {
            steps {
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} build --no-cache'
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} up -d'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Checking container status..."
                    docker ps | grep ${DOCKER_COMPOSE_PROJECT}
                    
                    echo "Waiting for services to start..."
                    sleep 15
                    
                    echo "Testing backend health..."
                    curl -s http://localhost:4001/health || true
                    
                    echo "Testing frontend..."
                    curl -s http://localhost:82 || true
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
            sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} logs > docker-compose.log'
        }
        failure {
            echo 'Deployment failed!'
            sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} logs > docker-compose.log'
            sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} down || true'
        }
        always {
            archiveArtifacts artifacts: 'docker-compose.log', allowEmptyArchive: true
            cleanWs()
        }
    }
} 