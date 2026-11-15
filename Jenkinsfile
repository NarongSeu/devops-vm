
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
    }

    options {
        skipDefaultCheckout(true)
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(
            logRotator(
                numToKeepStr: '3',
                daysToKeepStr: '7'
            )
        )
    }

    stages {

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

        stage('Git Checkout') {
            steps {
                echo 'Cloning GitHub Repository...'

                git branch: 'main',
                    url: 'https://github.com/NarongSeu/devops-vm.git'
            }
        }

        stage('Run Test') {
            when {
                expression {
                    params.RUN_TEST == true
                }
            }
            steps {
                sh '''
                    npm install
                    npm run test
                '''
            }
        }

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
                    '''
                }
            }
        }

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
            echo 'SonarQube: https://sonarqube.lorvengroth.cfd'
            echo 'Application: http://3.25.79.190:3005'
        }

        failure {
            echo 'FAILED: Check Jenkins Console Output!'
        }
    }
}
