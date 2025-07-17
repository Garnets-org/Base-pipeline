pipeline {
    agent { label 'docker' }

    parameters {
        string(name: 'REPO_URL', description: 'Git repo URL')
        string(name: 'DOCKER_IMAGE', description: 'Full image name with tag')
        string(name: 'DOCKER_CREDENTIALS_ID', defaultValue: 'dockerhub-creds', description: 'DockerHub credentials')
        string(name: 'APP_ENV', defaultValue: 'production', description: 'Deployment environment')
    }

    environment {
        IMAGE = "${params.DOCKER_IMAGE}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo " Cloning repo: ${params.REPO_URL}"
                git url: "${params.REPO_URL}", branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo " Building Docker image: ${IMAGE}"
                sh "docker build -t ${IMAGE} ."
            }
        }

        stage('Security Scan') {
            steps {
                echo " Running Snyk and Trivy scans..."
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    sh """
                        snyk auth \$SNYK_TOKEN
                        snyk test --docker ${IMAGE} --file=Dockerfile --severity-threshold=high
                    """
                }
                sh "trivy image ${IMAGE} || true"
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${params.DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    sh """
                        echo \$PASSWORD | docker login -u \$USERNAME --password-stdin
                        docker push ${IMAGE}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Simulated deploy to ${params.APP_ENV}"
            }
        }
    }
}

