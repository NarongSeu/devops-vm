
pipeline {
    agent any

    tools {
        nodejs 'nodejs-v22'
    }

    // Automatically build when GitHub receives a push
    triggers {
        githubPush()
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
        APP_PORT        = '3005'
        CONTAINER_PORT  = '3000'
    }

    options {
        skipDefaultCheckout(true)

        timeout(
            time: 15,
            unit: 'MINUTES'
        )

        buildDiscarder(
            logRotator(
                numToKeepStr: '3',
                daysToKeepStr: '7'
            )
        )

        disableConcurrentBuilds()
    }

    stages {

        // ==========================================
        // 1. TEST INSTALLED TOOLS
        // ==========================================

        stage('Test Tools') {
            steps {
                echo 'Checking installed tools...'

                sh '''
                    node --version
                    npm --version
                    docker --version
                    git --version
                    java -version
                '''
            }
        }

        // ==========================================
        // 2. GIT CHECKOUT
        // ==========================================

        stage('Git Checkout') {
            steps {
                echo 'Cloning GitHub Repository...'

                git branch: 'main',
                    url: 'https://github.com/NarongSeu/devops-vm.git'

                sh '''
                    echo "Current Git commit:"
                    git rev-parse --short HEAD
                '''
            }
        }

        // ==========================================
        // 3. RUN REACT TESTS
        // ==========================================

        stage('Run Test') {
            when {
                expression {
                    params.RUN_TEST == true
                }
            }

            steps {
                echo 'Running React application tests...'

                sh '''
                    npm install
                    npm run test
                '''
            }
        }

        // ==========================================
        // 4. SONARQUBE ANALYSIS
        // ==========================================

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
                            -Dsonar.exclusions=node_modules/**,build/**,dist/**,.scannerwork/**
                        """
                    }
                }
            }
        }

        // ==========================================
        // 5. QUALITY GATE
        // ==========================================

        stage('Quality Gate') {
            steps {
                timeout(
                    time: 5,
                    unit: 'MINUTES'
                ) {

                    script {

                        def qg = waitForQualityGate(
                            abortPipeline: false
                        )

                        echo "===================================="
                        echo "SONARQUBE QUALITY GATE RESULT"
                        echo "===================================="
                        echo "Status: ${qg.status}"
                        echo "===================================="

                        if (qg.status != 'OK') {

                            error(
                                "QUALITY GATE FAILED: ${qg.status}. " +
                                "Please check SonarQube dashboard."
                            )
                        }

                        echo 'QUALITY GATE PASSED!'
                    }
                }
            }
        }

        // ==========================================
        // 6. BUILD DOCKER IMAGE
        // ==========================================

        stage('Build Docker Image') {
            steps {
                echo 'Building React Docker Image...'

                sh '''
                    docker build \
                        -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} .

                    docker tag \
                        ${DOCKER_HUB_REPO}:${IMAGE_TAG} \
                        ${DOCKER_HUB_REPO}:latest

                    echo "Docker images created:"
                    docker images ${DOCKER_HUB_REPO}
                '''
            }
        }

        // ==========================================
        // 7. PUSH IMAGE TO DOCKER HUB
        // ==========================================

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
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}

                        docker push ${DOCKER_HUB_REPO}:latest

                        echo "Docker images pushed successfully!"
                    '''
                }
            }
        }

        // ==========================================
        // 8. DEPLOY REACT APPLICATION
        // ==========================================

        stage('Deploy') {
            steps {
                echo 'Deploying React Application...'

                sh '''
                    # Stop previous container
                    docker stop ${CONTAINER_NAME} || true

                    # Remove previous container
                    docker rm ${CONTAINER_NAME} || true

                    # Start new container
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:${CONTAINER_PORT} \
                        --restart unless-stopped \
                        ${DOCKER_HUB_REPO}:${IMAGE_TAG}

                    echo "Checking running containers..."
                    docker ps
                '''
            }
        }

        // ==========================================
        // 9. VERIFY APPLICATION
        // ==========================================

        stage('Verify Application') {
            steps {
                echo 'Verifying React Application...'

                sh '''
                    echo "Waiting for application to become ready..."

                    for i in $(seq 1 15); do

                        if curl -fsS http://localhost:${APP_PORT}/ \
                            -o /dev/null; then

                            echo "Application is running successfully!"
                            exit 0
                        fi

                        echo "Attempt $i: Application not ready yet..."
                        sleep 2
                    done

                    echo "Application verification failed!"

                    docker logs ${CONTAINER_NAME}

                    exit 1
                '''
            }
        }
    }

    // ==========================================
    // 10. POST BUILD ACTIONS
    // ==========================================

    post {

        always {
            echo 'Cleaning Jenkins workspace...'

            sh 'docker logout || true'

            cleanWs()
        }

        success {
            echo '===================================='
            echo 'SUCCESS: PIPELINE COMPLETED!'
            echo '===================================='

            echo 'SonarQube: https://sonarqube.lorvengroth.cfd'

            echo 'Application: http://3.25.79.190:3005'

            echo "Docker Image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
        }

        failure {
            echo '===================================='
            echo 'FAILED: PIPELINE EXECUTION!'
            echo '===================================='

            echo 'Please check Jenkins Console Output.'
        }
    }
}
