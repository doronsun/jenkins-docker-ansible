pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds') // ניצור את זה בג'נקינס עוד רגע
	IMAGE_NAME = "doronsun/myflaskapp"
    }

    stages {
        stage('Checkout') {
            steps {
                echo '🔍 Cloning source code...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                echo '🔐 Logging in to DockerHub...'
                sh 'echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin'
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo '📤 Pushing image to DockerHub...'
                sh 'docker push $IMAGE_NAME:latest'
            }
        }

        stage('Deploy with Ansible') {
            steps {
                echo '🚀 Deploying with Ansible...'
                sh 'ansible-playbook -i inventory deploy-playbook.yml'
            }
        }
    }
}

