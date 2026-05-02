pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = "puremahadi" 
        IMAGE_NAME = "dotnet-minimal-api"
        // এখানে V এর সাথে বিল্ড নম্বর যুক্ত করা হয়েছে
        DOCKER_IMAGE = "${DOCKER_HUB_USER}/${IMAGE_NAME}:V${BUILD_NUMBER}"
        
        DOCKER_HUB_CREDS_ID = 'docker-hub-creds'
        KUBE_CONFIG_CREDS_ID = 'kube-config'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // আপনার নতুন ব্রাঞ্চের নাম এখানে দেওয়া হয়েছে
                git branch: 'jenkins-CICD', url: 'https://github.com/puremahadi/DevOps.NET-minimal-API.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                // আপনার Dockerfile যে লোকেশনে আছে সেটি ব্যবহার করবে
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}"
                        
                        // 'latest' হিসেবেও পুশ করা যাতে সবসময় আপডেট ভার্সন পাওয়া যায়
                        sh "docker tag ${DOCKER_IMAGE} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig([credentialsId: "${KUBE_CONFIG_CREDS_ID}"]) {
                        // k8s ফোল্ডারের ভেতরের ফাইলগুলো অ্যাপ্লাই করা
                        sh "kubectl apply -f k8s/mssql-deployment.yaml"
                        sh "kubectl apply -f k8s/mssql-service.yaml"
                        sh "kubectl apply -f k8s/deployment.yaml"
                        sh "kubectl apply -f k8s/service.yaml"
                        
                        // ডাইনামিক ইমেজ আপডেট
                        sh "kubectl set image deployment/dotnet-minimal-api-deployment dotnet-api-container=${DOCKER_IMAGE}"
                        
                        // রোলআউট চেক
                        sh "kubectl rollout status deployment/dotnet-minimal-api-deployment"
                    }
                }
            }
        }
    }

    post {
        always {
            // ক্লিনআপ
            sh "docker rmi ${DOCKER_IMAGE} || true"
            sh "docker rmi ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest || true"
        }
    }
}