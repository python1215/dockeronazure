pipeline {
    agent any

    environment {
        IMAGE = "msmengr/demo-app:latest"
        SSH_USER = "azureuser"
        SSH_HOST = "20.81.45.129"
    }

    stages {

        stage("Checkout") {
            steps {
                git 'https://github.com/Mudassir-dev0ps/dockeronazure'
            }
        }

        stage("Build Docker Image") {
            steps {
                sh "docker build -t $IMAGE ."
            }
        }

        stage("Push Image to Docker Hub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push $IMAGE
                    """
                }
            }
        }

        stage("Deploy to Azure VM") {
            steps {
                sshagent(credentials: ['azure-vm-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no $SSH_USER@$SSH_HOST "
                            docker pull $IMAGE &&
                            docker stop demo || true &&
                            docker rm demo || true &&
                            docker run -d --name demo -p 80:5000 $IMAGE
                        "
                    """
                }
            }
        }
    }
}
