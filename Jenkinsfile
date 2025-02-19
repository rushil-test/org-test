pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "hello-cicd-app"
        ARTIFACTORY_REPO_KEY = "docker"
        ARTIFACTORY_PUSH_URL = "a8c6bcd2c5f69423a8910895a5c5a42c-237309825.us-west-2.elb.amazonaws.com/docker"
        HOST_PORT = "5000"  // Change this to your desired host port
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    git branch: 'main',  
                        credentialsId: '363533d8-d4a8-477c-855c-f9ca3dc86d0b',  
                        url: 'https://github.com/Rushil08/test_app.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def buildNumber = env.BUILD_NUMBER ?: sh(returnStdout: true, script: 'date +%s').trim()
                    def imageNameWithTag = "${ARTIFACTORY_PUSH_URL}/${DOCKER_IMAGE_NAME}:${buildNumber}"
                    env.IMAGE_NAME_WITH_TAG = imageNameWithTag

                    sh "docker build -t ${imageNameWithTag} ."
                }
            }
        }

        stage('Publish to Artifactory') {
            steps {
                script {
                    rtDockerPush(
                        serverId: "artifactory-server",
                        image: env.IMAGE_NAME_WITH_TAG,
                        targetRepo: ARTIFACTORY_REPO_KEY,
                        buildName: env.JOB_NAME,
                        buildNumber: env.BUILD_NUMBER
                    )
                }
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    sh "docker pull ${env.IMAGE_NAME_WITH_TAG}"
                    sh "docker stop ${DOCKER_IMAGE_NAME} || true"
                    sh "docker rm ${DOCKER_IMAGE_NAME} || true"
                    sh "docker run -d --name ${DOCKER_IMAGE_NAME} -p ${HOST_PORT}:8080 ${env.IMAGE_NAME_WITH_TAG}"
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    sh "docker rmi ${env.IMAGE_NAME_WITH_TAG} || true"
                }
            }
        }
    }
}