pipeline {
	agent any

	environment {
        IMAGE_MAIN = "nodemain:v1.0"
        IMAGE_DEV  = "nodedev:v1.0"
        PORT_MAIN  = "3000"
        PORT_DEV   = "3001"
    }

	tools {
        nodejs 'node'
        dockerTool 'docker'
    }

	stages {
		stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building Node.js app..."
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo "Running tests..."
                sh 'npm test'
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    def imageName = (env.BRANCH_NAME == 'main') ? env.IMAGE_MAIN : env.IMAGE_DEV
                    echo "Building Docker image: ${imageName}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Deploy') {
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
