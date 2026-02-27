pipeline {

    agent none 

    environment {
        PATH           = "/root/.bun/bin:${env.PATH}"
        APP_NAME       = 'jenkins-app'
        IMAGE_NAME     = "rovidev/${APP_NAME}"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        CONTAINER_PORT = '3000'
        HOST_PORT      = '3000'
    }

    stages {

        // ================== CI — corre dentro del contenedor node:20-alpine ==================
        stage('CI') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            stages {
                stage('Install') {
                    steps { 
                        echo '================== [CI] Installing dependencies =================='
                        sh 'bun install' 
                    }
                }
                stage('Build') {
                    steps { 
                        echo '================== [CI] Building application =================='
                        sh 'bun run build' 
                    } 
                }

                stage('Docker Build & Push') {
                    
                    steps {
                        echo '================== [CI] Building and pushing Docker image =================='
                        withCredentials([usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            sh """
                                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                                docker push ${IMAGE_NAME}:latest
                            """
                        }
                    }
                }
            }
            
        }


        // ====== CD — corre directamente en Jenkins, fuera de cualquier contenedor ======

        stage('CD') {
            agent any  // ← corre en Jenkins directamente, no en docker
            steps {
                echo '================== [CD] Deploying application =================='
                sh """
                    docker stop ${APP_NAME} || true
                    docker rm   ${APP_NAME} || true

                    docker run -d \
                        --name ${APP_NAME} \
                        --restart unless-stopped \
                        -p ${HOST_PORT}:${CONTAINER_PORT} \
                        ${IMAGE_NAME}:latest
                """
            }
        }

    }

    post {
        always {
            echo '================== Pipeline Execution Completed =================='
        }
        success {
            echo "================== Build #${BUILD_NUMBER} deployed successfully =================="
        }
        failure {
            echo "================== Build #${BUILD_NUMBER} failed =================="
        }
    }
}
