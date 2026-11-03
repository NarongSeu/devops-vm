// pipeline {
//     agent any

//     stages {
//         stage("Build"){
//             steps{
//                 sh """
//                     docker build -t jenkins-react-pipeline . 
//                 """
//             }
//         }


//         stage("Deploy"){
//             steps{
//                 sh"""
                
//                  docker stop reactjs-cont || true 
//                 docker rm reactjs-cont || true 
                
//                 docker run -dp 3000:80 \
//                     --name reactjs-cont \
//                     jenkins-react-pipeline
//                 """

//             }
//         }
    
//         stage("Add Domain name "){
//             steps{
//                 sh """
//                 echo "Runing shellscript to add the domain name for the service " 
                
//                 """
//             }
//         }
        
//     }
// }
pipeline {
    agent any
    
    environment {
        // Updated to match your exact Docker Hub repository name
        DOCKER_HUB_REPO = 'narongseu09030903/devsolve'
        // Jenkins Credential ID configured in Jenkins Dashboard
        CREDENTIALS_ID = 'dockerhub-credentials'
    }

    stages {
        // Stage 1: Git Clone
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/keoKAY/reactjs-devop8-template.git'[cite: 1]
            }
        }

        // Stage 2: Build Docker Image using Layered Dockerfile
        stage('Build') {
            steps {
                sh """
                    docker build -t ${DOCKER_HUB_REPO}:${env.BUILD_NUMBER} -t ${DOCKER_HUB_REPO}:latest .
                """
            }
        }

        // Stage 3: Push to DockerHub with Jenkins Credentials Injection
        stage('Push') {
            steps {
                script {
                    echo "Injecting Jenkins Credentials and pushing to DockerHub..."
                    withCredentials([usernamePassword(credentialsId: "${CREDENTIALS_ID}", 
                                                      usernameVariable: 'DOCKER_USER', 
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}"
                        sh "docker push ${DOCKER_HUB_REPO}:latest"
                    }
                }
            }
        }

        // Stage 4: Deploy (Mapping host port 3005 to container port 3000)
        stage('Deploy') {
            steps {
                sh """
                    docker stop reactjs-cont || true 
                    docker rm reactjs-cont || true 
                    
                    docker run -d -p 3005:3000 \
                        --name reactjs-cont \
                        ${DOCKER_HUB_REPO}:latest
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
        success {
            echo "Pipeline executed successfully! Application is running on port 3005."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}