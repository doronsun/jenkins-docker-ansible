pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "doronsun/myflaskapp"
    }

    parameters {
        choice(name: 'SERVICE', choices: ['service1', 'service2'], description: 'Select service to deploy')
        string(name: 'TAG', defaultValue: 'latest', description: 'Docker image tag')
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
                script {
                    env.FULL_IMAGE_NAME = "${IMAGE_NAME}-${params.SERVICE}:${params.TAG}"
                    sh "docker build -t ${env.FULL_IMAGE_NAME} ./${params.SERVICE}"
                }
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
                sh "docker push ${env.FULL_IMAGE_NAME}"
            }
        }

        stage('Deploy with Ansible') {
            steps {
                echo '🚀 Deploying with Ansible...'
                script {
                    def containerPort = params.SERVICE == 'service1' ? '5000' : '5001'
                    sh """
                        ansible-playbook -i inventory deploy-playbook.yml \\
                        -e "service_name=${params.SERVICE}" \\
                        -e "docker_image=${env.FULL_IMAGE_NAME}" \\
                        -e "container_port=${containerPort}"
                    """
                }
            }
        }
    }
}

