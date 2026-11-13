
pipeline {
    agent any

    tools {
        nodejs 'nodejs-v22'
    }

    parameters {
        booleanParam(
            name: 'RUN_TEST',
            defaultValue: false,
            description: 'Run React application tests'
        )
    }

    environment {
        DOCKER_HUB_REPO = 'narongseu09030903/devsolve'
        IMAGE_TAG       = "${BUILD_NUMBER}"
        DOCKER_CREDS    = 'dockerhub-credentials'
        CONTAINER_NAME  = 'reactjs-cont'
    }

    options {
        // Keep the latest 3 builds or builds from the last 7 days
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(
            logRotator(
                numToKeepStr: '3',
                daysToKeepStr: '7'
            )
        )
    }

    stages {

        // Stage 1: Call another Jenkins job
        stage('Call Other Job') {
            steps {
                build 'first-demo'
            }
        }

        // Stage 2: Check tools
        stage('Test Tools') {
            steps {
                sh '''
                    echo "Checking installed tools..."
                    node --version
                    npm --version
                    docker --version
                    git --version
                    java -version
                '''
            }
        }

        // Stage 3: Git Clone
        stage('Git Clone') {
            steps {
                echo 'Cloning GitHub Repository...'

                git branch: 'main',
                    url: 'https://github.com/NarongSeu/devops-vm.git'
            }
        }

        // Stage 4: Run Tests (Optional)
        stage('Run Test') {
            when {
                expression {
                    params.RUN_TEST == true
                }
            }
            steps {
                sh '''
                    echo "Installing dependencies..."
                    npm install

                    echo "Running tests..."
                    npm run test
                '''
            }
        }

        // Stage 5: SonarQube Analysis
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQube'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=devops-vm \
                            -Dsonar.projectName=devops-vm \
                            -Dsonar.sources=. \
                            -Dsonar.sourceEncoding=UTF-8 \
                            -Dsonar.exclusions=node_modules/**,build/**,dist/**
                        """
                    }
                }
            }
        }

        // Stage 6: Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate(
                            abortPipeline: true
                        )

                        echo "Quality Gate Status: ${qg.status}"
                    }
                }
            }
        }

        // Stage 7: Build Docker Image
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image...'

                sh '''
                    docker build \
                        -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} .

                    docker tag \
                        ${DOCKER_HUB_REPO}:${IMAGE_TAG} \
                        ${DOCKER_HUB_REPO}:latest
                '''
            }
        }

        // Stage 8: Push Docker Hub
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

        // Stage 9: Deploy
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

        // Stage 10: Verify Application
        stage('Verify Application') {
            steps {
                sh '''
                    echo "Waiting for application..."
                    sleep 5

                    curl -f http://localhost:3005
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
            cleanWs()
        }

        success {
            echo 'SUCCESS: Pipeline executed successfully!'
            echo 'SonarQube: http://3.25.79.190:9000'
            echo 'Application: http://3.25.79.190:3005'
        }

        failure {
            echo 'FAILED: Check Jenkins Console Output!'
        }
    }
}
