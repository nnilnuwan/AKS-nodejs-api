pipeline {
    agent any 

    environment {
        DOCKER_IMAGE = "nilnuwan/nodejs-api"
        VERSION_FILE = 'VERSION.txt'  // File where the version is stored
        VERSION = ''  // Initialize VERSION variable
    }

    stages {
        // Stage to fetch the version and increment it
        stage('Increment Version') {
            steps {
                script {
                    // Check if VERSION.txt exists, else initialize version as 1.0.0
                    if (fileExists(VERSION_FILE)) {
                        VERSION = readFile(VERSION_FILE).trim()
                    } else {
                        VERSION = '1.0.0'  // Initial version if no VERSION.txt file exists
                    }

                    // Split the version into parts (major.minor.patch)
                    def versionParts = VERSION.split('\\.')

                    // Increment the patch version (you can change this to minor/major if needed)
                    versionParts[2] = (versionParts[2].toInteger() + 1).toString()

                    // Join the version parts back into a string
                    def newVersion = versionParts.join('.')

                    // Save the new version to VERSION.txt
                    writeFile file: VERSION_FILE, text: newVersion
                    env.VERSION = newVersion  // Set the new version as an environment variable
                }
            }
        }

        // Stage to checkout the source code
        stage('SCM Checkout') {
            steps {
                retry(3) {
                    git branch: 'main', url: 'https://github.com/nnilnuwan/AKS-nodejs-api'
                }
            }
        }

        // Stage to build the Docker image with the incremented version
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${VERSION} ."
            }
        }

        // Stage to login to Docker Hub
        stage('Login to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-cred', variable: 'docker-cred')]) {
                    script {
                        sh "docker login -u nilnuwan -p Sanka@2002"
                    }
                }
            }
        }

        // Stage to push the Docker image to Docker Hub
        stage('Push Image to Docker Hub') {
            steps {
                sh "docker push ${DOCKER_IMAGE}:${VERSION}"
            }
        }

        // Stage to deploy the Docker image to AKS cluster
        stage("Deploy to AKS cluster") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                    sh "kubectl set image deployment/nodejs-api-deployment nodejs-api-container=${DOCKER_IMAGE}:${VERSION} -n nodejs-api"
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

