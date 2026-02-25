pipeline {

    agent any

    environment {
        PATH = "$PATH:/root/.bun/bin"
    }

    stages {

        stage('Setup & Build') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo '========== Installing dependencies =========='
                }
                sh '''
                    npm install -g bun
                    bun install
                    bun run build
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo '========== Deploy build =========='
                }
                sh '''
                    bun install -g serve
                    serve -s dist -l 3000 &
                '''
            }
        }

    }

    post {
        always {
            script {
                echo '========== Pipeline Execution Completed =========='
            }
        }
        success {
            script {
                echo '========== Build Successful =========='
            }
        }
        failure {
            script {
                echo '========== Build Failed =========='
            }
        }
    }
}