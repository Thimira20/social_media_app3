pipeline {
    agent any

    environment {
        // Docker Hub credentials (configured in Jenkins)
        DOCKER_CREDENTIALS_ID = 'docker-password'
        DOCKER_REPO_FE = 'thimira20/session-react-frontend'
        DOCKER_REPO_BE = 'thimira20/session-node-backend'

        // AWS credentials and instance details
        AWS_CREDENTIALS_ID = 'aws-credentials'
        SSH_KEY_ID = 'ec2-ssh-key'
        EC2_HOST = '18.212.30.151'
        EC2_USER = 'ubuntu' // or your EC2 instance user
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Thimira20/social_media_app3.git', branch: 'main'
            }
        }

        stage('Build Frontend') {
            steps {
                dir('client') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'npm install'
                }
            }
        }

        stage('Dockerize and Push Frontend') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                        sh """
                        docker build -t ${DOCKER_REPO_FE}:latest ./client
                        docker push ${DOCKER_REPO_FE}:latest
                        """
                    }
                }
            }
        }

        stage('Dockerize and Push Backend') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                        sh """
                        docker build -t ${DOCKER_REPO_BE}:latest ./backend
                        docker push ${DOCKER_REPO_BE}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy on AWS EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: SSH_KEY_ID, keyFileVariable: 'EC2_SSH_KEY')]) {
                    sh """
                    ssh -i ${EC2_SSH_KEY} ${EC2_USER}@${EC2_HOST} << EOF
                        docker pull ${DOCKER_REPO_FE}:latest
                        docker pull ${DOCKER_REPO_BE}:latest
                        docker stop react-frontend || true
                        docker stop node-backend || true
                        docker rm react-frontend || true
                        docker rm node-backend || true
                        docker run -d --name react-frontend -p 80:80 ${DOCKER_REPO_FE}:latest
                        docker run -d --name node-backend -p 4000:4000 ${DOCKER_REPO_BE}:latest
                    EOF
                    """
                }
            }
        }
    }

    post {
        always {
           echo "pipeline completed"
            
        }
    }
}
