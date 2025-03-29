pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "anand1827/gradle_app"
        EC2_USER = "ec2-user"
        EC2_HOST = "43.204.115.104"
        DOCKER_HUB_CREDENTIALS = "docker-hub-credentials"
        SSH_CREDENTIALS = "ec2-ssh-key"
        GIT_REPO = "https://github.com/Anand1827/Udemy_project.git"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: "${GIT_REPO}"
            }
        }

        stage('Build with Gradle') {
            steps {
                sh './gradlew clean build'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: SSH_CREDENTIALS, keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i $SSH_KEY $EC2_USER@$EC2_HOST << EOF
                            docker pull ${DOCKER_IMAGE}
                            docker stop gradle_app || true
                            docker rm gradle_app || true
                            docker run -d --name gradle_app -p 8080:8080 ${DOCKER_IMAGE}
                            EOF
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
