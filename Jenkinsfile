pipeline {
	agent any
	environment {
		IMAGE_MAIN = "nodemain:v1.0"
		IMAGE_DEV = "nodedev:v1.0"
	}

	stages {
		stage('Checkout') {
			steps {
				checkout scm
			}
		}

		stage('Build') {
            steps {
                echo "Installing dependencies..."
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
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker build -t ${IMAGE_MAIN} ."
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh "docker build -t ${IMAGE_DEV} ."
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Stop and remove old containers ONLY from the node apps
                    sh """
                    docker ps -q --filter "ancestor=nodemain:v1.0" | xargs -r docker stop
                    docker ps -q --filter "ancestor=nodedev:v1.0" | xargs -r docker stop
                    docker ps -a -q --filter "ancestor=nodemain:v1.0" | xargs -r docker rm
                    docker ps -a -q --filter "ancestor=nodedev:v1.0" | xargs -r docker rm
                    """

                    echo "Running new container..."
                    if (env.BRANCH_NAME == 'main') {
                        sh 'docker run -d --expose 3000 -p 3000:3000 nodemain:v1.0'
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh 'docker run -d --expose 3001 -p 3001:3000 nodedev:v1.0'
                    }
                }
            }
        }
	}
}
