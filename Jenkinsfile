
pipeline {
    agent any

    tools {
        nodejs 'nodejs-v22'
    }

    environment {
        DOCKER_HUB_REPO = 'narongseu09030903/devsolve'
        IMAGE_TAG       = "${BUILD_NUMBER}"
        DOCKER_CREDS    = 'dockerhub-credentials'
        CONTAINER_NAME  = 'reactjs-cont'
    }

    stages {

        // Stage 1: Check Tools
        stage('Test Tools') {
            steps {
                sh '''
                    echo "Checking NodeJS..."
                    node --version
                    npm --version
                    docker --version
                '''
            }
        }

        // Stage 2: Git Clone
        stage('Git Clone') {
            steps {
                echo 'Cloning GitHub Repository...'

                git branch: 'main',
                    url: 'https://github.com/NarongSeu/devops-vm.git'
            }
        }

        stage("Run test"){
            steps{
                sh"""
                    npm install 
                    npm run test
                """
            }
        }

        // Stage 3: Build Docker Image
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image...'

                sh '''
                    docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} .
                    docker tag ${DOCKER_HUB_REPO}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:latest
                '''
            }
        }

        // Stage 4: Push Docker Hub
        stage('Push Docker Hub') {
            steps {
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

        // Stage 5: Deploy
        stage('Deploy') {
            steps {
                echo 'Deploying React Application...'

                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 3005:3000 \
                        --restart unless-stopped \
                        ${DOCKER_HUB_REPO}:latest

                    docker ps
                '''
            }
        }

        // Stage 6: Verify
        stage('Verify Application') {
            steps {
                sh '''
                    sleep 5
                    curl -f http://localhost:3005
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }

        success {
            echo 'SUCCESS: Pipeline executed successfully!'
            echo 'Application: http://YOUR_SERVER_IP:3005'
        }

        failure {
            echo 'FAILED: Check Jenkins Console Output!'
        }
    }
}
