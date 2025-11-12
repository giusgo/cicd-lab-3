pipeline {
	agent none

	environment {
        IMAGE_MAIN = "nodemain:v1.0"
        IMAGE_DEV  = "nodedev:v1.0"
        PORT_MAIN  = "3000"
        PORT_DEV   = "3001"
    }

	tools {
        nodejs 'node'
    }

	stages {
		stage('Checkout') {
            agent any
            steps {
                checkout scm
            }
        }

        stage('Build') {
            agent { label 'node' }
            tools { nodejs 'node' }
            steps {
                echo "Building Node.js app..."
                sh 'npm install'
            }
        }

        stage('Test') {
            agent { label 'node' }
            tools { nodejs 'node' }
            steps {
                echo "Running tests..."
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            agent { label 'docker' }
            steps {
                script {
                    def imageName = (env.BRANCH_NAME == 'main') ? env.IMAGE_MAIN : env.IMAGE_DEV
                    echo "Building Docker image: ${imageName}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Deploy') {
            agent { label 'docker' }
            steps {
                script {
                    def port = (env.BRANCH_NAME == 'main') ? env.PORT_MAIN : env.PORT_DEV
                    def imageName = (env.BRANCH_NAME == 'main') ? env.IMAGE_MAIN : env.IMAGE_DEV

                    echo "Deploying ${imageName} on port ${port}..."

                    // stop and remove old containers (only matching image)
                    sh "docker ps -q --filter ancestor=${imageName} | xargs -r docker stop"
                    sh "docker ps -aq --filter ancestor=${imageName} | xargs -r docker rm"

                    // run new container
                    sh "docker run -d -p ${port}:3000 ${imageName}"
                }
            }
        }
	}
}
