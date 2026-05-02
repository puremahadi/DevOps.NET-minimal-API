pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = "puremahadi" 
        IMAGE_NAME = "dotnet-minimal-api"
        DOCKER_IMAGE = "${DOCKER_HUB_USER}/${IMAGE_NAME}:V${BUILD_NUMBER}"
        
        DOCKER_HUB_CREDS_ID = 'docker-hub-creds'
        KUBE_CONFIG_CREDS_ID = 'kube-config'
    }

    stages {
        stage('🚚 SCM Cleanup & Checkout') {
            steps {
                // পুরানো ফাইল ক্লিন করে নতুন করে কোড নামানো
                cleanWs()
                git branch: 'jenkins-CICD', url: 'https://github.com/puremahadi/DevOps.NET-minimal-API.git'
            }
        }

        stage('🏗️ Build Container Image') {
            steps {
                script {
                    echo "Building Version: V${BUILD_NUMBER}..."
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('🚀 Publish to Docker Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}"
                        sh "docker tag ${DOCKER_IMAGE} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('☸️ K8s Cluster Deployment') {
            steps {
                script {
                    withKubeConfig([credentialsId: "${KUBE_CONFIG_CREDS_ID}"]) {
                        // ম্যানিফেস্ট অ্যাপ্লাই এবং ইমেজ আপডেট
                        sh "kubectl apply -f k8s/" 
                        sh "kubectl set image deployment/dotnet-minimal-api-deployment dotnet-api-container=${DOCKER_IMAGE}"
                        
                        echo "Deployment Successful for V${BUILD_NUMBER}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Please check logs."
        }
        always {
            // ক্লিনআপ স্টেজ
            sh "docker rmi ${DOCKER_IMAGE} || true"
        }
    }
}