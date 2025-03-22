pipeline {
    agent any
    environment {
        // Define image names
        BACKEND_IMAGE = "iamjabastin/docker-backend:latest"
        FRONTEND_IMAGE = "iamjabastin/docker-frontend:latest"
        // Define container names
        BACKEND_CONTAINER = "docker-running-backend"
        FRONTEND_CONTAINER = "docker-running-frontend"
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-jabastin', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/iamJABASTIN/DevOps-day-5.git", branch: 'main'
                }
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                // Build the backend image using the Dockerfile in the backend folder.
                sh 'docker build -t $BACKEND_IMAGE -f backend/dockerfile .'
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                // Build the frontend image using the Dockerfile in the frontend folder.
                sh 'docker build -t $FRONTEND_IMAGE -f frontend/dockerfile .'
            }
        }

        stage('Login to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-jabastin', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh 'docker push $BACKEND_IMAGE'
                sh 'docker push $FRONTEND_IMAGE'
            }
        }

        stage('Stop & Remove Existing Containers') {
            steps {
                script {
                    sh '''
                    if [ "$(docker ps -aq -f name=$BACKEND_CONTAINER)" ]; then
                        docker stop $BACKEND_CONTAINER || true
                        docker rm $BACKEND_CONTAINER || true
                    fi
                    if [ "$(docker ps -aq -f name=$FRONTEND_CONTAINER)" ]; then
                        docker stop $FRONTEND_CONTAINER || true
                        docker rm $FRONTEND_CONTAINER || true
                    fi
                    '''
                }
            }
        }

        stage('Run Docker Containers') {
            steps {
                // Run backend container (exposing port 5000)
                sh 'docker run -d -p 5000:5000 --name $BACKEND_CONTAINER $BACKEND_IMAGE'
                // Run frontend container (exposing port 5001 mapped to container port 80)
                sh 'docker run -d -p 5001:80 --name $FRONTEND_CONTAINER $FRONTEND_IMAGE'
            }
        }
    }

    post {
        success {
            echo "Build, push, and container execution successful!"
        }
        failure {
            echo "Build or container execution failed."
        }
    }
}
