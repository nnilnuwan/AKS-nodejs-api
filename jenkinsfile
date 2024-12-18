pipeline {
    agent any 
    
    environment {
        DOCKER_IMAGE = "nilnuwan/nodejs-api:LTS6"
    }

    stages { 
        stage('SCM Checkout') {
            steps {
                retry(3) {
                    git branch: 'main', url: 'https://github.com/nnilnuwan/AKS-nodejs-api'
                }
            }
        }

        stage('Build Docker Image') {
            steps {  
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-cred', variable: 'docker-cred')]) {
                    script {
                        sh "docker login -u nilnuwan -p Sanka@2002"
                    }
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage ("Deploy to AKS cluster") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                    sh "kubectl apply -f node-api-deployment.yml -n nodejs-api"
                // some block
                }
            }
        }
    }

    post {
        always {
            // Ensure to logout from Docker Hub after the job finishes
            sh 'docker logout'
        }
    }
}

