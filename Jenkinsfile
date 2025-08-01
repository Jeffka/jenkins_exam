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
                        sh '''
                        docker build -t ${DOCKERHUB_USER}/k-cast-service:latest ./cast-service
                        docker push ${DOCKERHUB_USER}/k-cast-service:latest
                        ''' 
                        }
                    }
                }
                stage('movie-service') {
                    steps {
                        script {
                        sh '''
                        docker build -t ${DOCKERHUB_USER}/k-movie-service:latest ./movie-service
                        docker push ${DOCKERHUB_USER}/k-movie-service:latest
                        '''
                        }
                    }
                }
            }
        }
            
        stage('Deploy to dev') {
            steps {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp jenkinsexam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install jenkins-dev ./jenkinsexam --namespace dev --create-namespace
                '''
            }
        }
        stage('Deploy to staging') {
            steps {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp jenkinsexam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install jenkins-staging ./jenkinsexam --namespace staging --create-namespace
                '''
            }
        }
        stage('Deploy to qa') {
            steps {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp jenkinsexam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install jenkins-qa ./jenkinsexam --namespace qa --create-namespace
                '''
            }
        }
            
        
        stage('Deploy to prod') {
            environment {
                KUBECONFIG = credentials('config')
            }
            when {
                branch 'main'
            }
                steps {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp jenkinsexam/values.yaml values.yml
                        cat values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install jenkins-prod ./jenkinsexam --namespace prod --create-namespace
                        '''
                }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
