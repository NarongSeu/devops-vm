
pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'narongseu09030903/devsolve'
        IMAGE_TAG       = "${BUILD_NUMBER}"
        DOCKER_CREDS    = 'dockerhub-credentials'
        CONTAINER_NAME  = 'reactjs-cont'
    }

    stages {

        // Stage 1: Git Clone
        stage('Git Clone') {
            steps {
                echo 'Cloning GitHub Repository...'

                git branch: 'main',
                    url: 'https://github.com/NarongSeu/devops-vm.git'
            }
        }

        // Stage 2: Build Docker Image
        stage('Build') {
            steps {
                echo 'Building Docker Image...'

                sh """
                    docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} .
                    docker tag ${DOCKER_HUB_REPO}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:latest
                """
            }
        }

        // Stage 3: Push to Docker Hub
        stage('Push Docker Hub') {
            steps {
                echo 'Pushing Image to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                        docker push ${DOCKER_HUB_REPO}:latest

                        docker logout
                    '''
                }
            }
        }

        // Stage 4: Deploy
        stage('Deploy') {
            steps {
                echo 'Deploying React Application...'

                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 3005:3000 \
                        --restart unless-stopped \
                        ${DOCKER_HUB_REPO}:latest
                """
            }
        }

        // Stage 5: Add Domain Name
        stage('Add Domain Name') {
            steps {
                echo 'Configuring Domain Name...'

                sh """
                    echo "Application is running on port 3005"
                    echo "Configure your domain DNS to point to your server IP."
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
            echo 'Pipeline executed successfully!'
            echo 'Application: http://YOUR_SERVER_IP:3005'
        }

        failure {
            echo 'Pipeline failed. Please check Console Output.'
        }
    }
}
