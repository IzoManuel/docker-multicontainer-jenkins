pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = credentials('DOCKERHUB_USERNAME')
        DOCKERHUB_ACCESS_TOKEN = credentials('DOCKERHUB_ACCESS_TOKEN')
    }

    stages {
        stage('Install Dependencies') {
            steps {
                script {
                    echo "Installing docker-compose"
                    sh 'apt-get update'
                    sh 'apt-get install -y docker-compose jq'
                    sh 'docker-compose --version'
                }
            }
        }

        stage('Pre-Build') {
            steps {
                script {
                    echo "Logging in to Docker Hub"
                    sh 'echo $DOCKERHUB_ACCESS_TOKEN | docker login -u $DOCKERHUB_USERNAME --password-stdin'
                    env.COMMIT_HASH = sh(script: 'echo $GIT_COMMIT | cut -c 1-7', returnStdout: true).trim()
                    echo "Commit hash: ${env.COMMIT_HASH}"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building Docker images"
                    sh 'docker-compose build'
                }
            }
        }

        stage('Post-Build') {
            steps {
                script {
                    echo "Tagging Docker images"
                    sh '''
                        docker tag jk-service-01:latest $DOCKERHUB_USERNAME/jk-service-01:$COMMIT_HASH
                        docker tag jk-service-02:latest $DOCKERHUB_USERNAME/jk-service-02:$COMMIT_HASH
                    '''
                    echo "Pushing Docker images to Docker Hub"
                    sh '''
                        docker push $DOCKERHUB_USERNAME/jk-service-01:$COMMIT_HASH
                        docker push $DOCKERHUB_USERNAME/jk-service-02:$COMMIT_HASH
                    '''
                }
            }
        }
    }
}
