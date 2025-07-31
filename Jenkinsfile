pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials'
        DOCKERHUB_USER = 'jeffka'
        IMAGE_TAG_LATEST = 'latest'
        IMAGE_TAG_COMMIT = "${GIT_COMMIT[0..6]}"
        IMAGE_TAG_BUILD = "build-${BUILD_ID}"
    }

    stages {
        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin'
                }
            }
        }

        stage('Build and Push Images') {
            parallel {
                stage('cast-service') {
                    steps {
                        script {
                        sh 'docker build -t ${DOCKERHUB_USER}/cast-service:${IMAGE_TAG_BUILD} ./cast-service' 
                        }
                    }
                }
                stage('movie-service') {
                    steps {
                        script {
                        sh 'docker build -t ${DOCKERHUB_USER}/movie-service:${IMAGE_TAG_BUILD} ./movie-service'
                        }
                    }
                }
            }
        }
        stage('Deploy non-prod environments') {
            parallel {
                stage('dev') {
                    steps {
                        sh 'helm upgrade --install jenkins-dev ./jenkinsexam --namespace dev --create-namespace'
                    }
            }
                stage('staging') {
                    steps {
                        sh 'helm upgrade --install jenkins-staging ./jenkinsexam --namespace staging --create-namespace'
                    }
                }
                stage('qa') {
                    steps {
                        sh 'helm upgrade --install jenkins-qa ./jenkinsexam --namespace qa --create-namespace'
                    }
                }
            }
        }
        stage('Deploy to prod') {
            when {
                branch 'main'
            }
                steps {
                    sh 'helm upgrade --install jenkins-prod ./jenkinsexam --namespace prod --create-namespace'
                }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
