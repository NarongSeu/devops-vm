pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'narongseu09030903/devsolve'
        IMAGE_TAG       = "${env.BUILD_NUMBER}"
        DOCKER_CREDS    = 'dockerhub-credentials'
        CONTAINER_NAME  = 'reactjs-cont'
    }

    stages {
        // Stage 1: Git Clone
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/NarongSeu/devops-vm.git'[cite: 1]
            }
        }

        // Stage 2: Build Docker Image
        stage('Build') {
            steps {
                echo "Building Docker Image..."
                sh "docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKER_HUB_REPO}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:latest"
            }
        }

        // Stage 3: Push to Docker Hub
        stage('Push Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_REPO}:latest"
                }
            }
        }

        // Stage 4: Deploy
        stage('Deploy') {
            steps {
                echo "Deploying Container on port 3005:3000..."
                sh """
                    # Stop and remove existing container if running
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    # Run new container mapping host port 3005 to container port 3000
                    docker run -d --name ${CONTAINER_NAME} -p 3005:3000 ${DOCKER_HUB_REPO}:latest
                """
            }
        }

        // Stage 5: Add Domain Name
        stage('Add Domain Name') {
            steps {
                sh """
                    echo "Running shell script to add the domain name for the service..."
                """
            }
        }
    }

    post {
        always {
            sh '''
                docker logout || true
                docker image prune -f || true
            '''
        }
        success {
            echo "Pipeline executed successfully! Application is running on port 3005."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}