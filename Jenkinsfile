pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = "puremahadi" 
        IMAGE_NAME = "devops-dotnet-simple-project" // আপনি এই নামটিই ব্যবহার করতে চেয়েছেন
        DOCKER_IMAGE = "${DOCKER_HUB_USER}/${IMAGE_NAME}:V${BUILD_NUMBER}"
        
        DOCKER_HUB_CREDS_ID = 'docker-hub-creds'
        KUBE_CONFIG_CREDS_ID = 'kube-config'
        GITHUB_CREDS_ID = 'my-vm-token' // জেনকিন্সে আপনার গিটের ক্রেডেনশিয়াল আইডি
    }

    stages {
        stage('🚚 Checkout & Cleanup') {
            steps {
                cleanWs()
                git branch: 'jenkins-CICD', url: 'https://github.com/puremahadi/DevOps.NET-minimal-API.git'
            }
        }

        stage('🏗️ Build & Tag Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
                sh "docker tag ${DOCKER_IMAGE} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('🚀 Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('📝 Update Manifest & Git Push') {
            steps {
                script {
                    // k8s/deployment.yaml ফাইলে নতুন ইমেজ ট্যাগ বসানো
                    sh "sed -i 's|image: ${DOCKER_HUB_USER}/.*|image: ${DOCKER_IMAGE}|g' k8s/deployment.yaml"
                    
                    // গিটহাবে পরিবর্তন পুশ করা
                    withCredentials([usernamePassword(credentialsId: "${GITHUB_CREDS_ID}", passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                        sh """
                            git config user.name 'jenkins-bot'
                            git config user.email 'jenkins@devops.com'
                            git add k8s/deployment.yaml
                            git commit -m "Update image to V${BUILD_NUMBER} [skip ci]"
                            git push https://${GIT_USER}:${GIT_PASS}@github.com/puremahadi/DevOps.NET-minimal-API.git jenkins-CICD
                        """
                    }
                }
            }
        }

        stage('☸️ Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig([credentialsId: "${KUBE_CONFIG_CREDS_ID}"]) {
                        // আপডেট হওয়া ফাইলটি অ্যাপ্লাই করা
                        sh "kubectl apply -f k8s/deployment.yaml"
                        // ফোর্স আপডেট নিশ্চিত করা
                        sh "kubectl set image deployment/dotnet-minimal-api-deployment dotnet-api-container=${DOCKER_IMAGE}"
                    }
                }
            }
        }
    }
}