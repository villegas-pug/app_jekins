// Pipeline declarativo de Jenkins - Automatización CI/CD con Docker
pipeline {
    // Define el entorno de ejecución del pipeline
    agent {
        // Utiliza Docker como agente en lugar del servidor Jenkins
        docker {
            // Imagen base: Node.js 20 en Alpine Linux (ligera y eficiente)
            image 'node:20-alpine'
            // Monta el socket de Docker para permitir comandos docker dentro del contenedor
            // args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    // Variables de entorno disponibles en todo el pipeline
    environment {
        // Indica que es entorno de producción
        NODE_ENV = 'production'
        // Flag que indica que se está ejecutando en CI/CD
        CI = 'true'
    }

    // Define los pasos del pipeline
    stages {
        // ====== STAGE 1: DESCARGAR CÓDIGO FUENTE ======
        stage('Checkout') {
            steps {
                // Imprime un separador visual en los logs
                script {
                    echo '========== Checking out source code =========='
                }
                // Checkout descarga el código del repositorio Git
                checkout scm
            }
        }

        // ====== STAGE 2: INSTALAR DEPENDENCIAS ======
        stage('Setup') {
            steps {
                // Imprime un separador visual en los logs
                script {
                    echo '========== Installing dependencies =========='
                }
                // Ejecuta comandos de shell en el contenedor
                sh '''
                    // Instala Bun globalmente (gestor de paquetes JavaScript rápido)
                    npm install -g bun
                    // Instala todas las dependencias del proyecto
                    bun install
                '''
            }
        }

        // ====== STAGE 3: VALIDAR CALIDAD DEL CÓDIGO ======
        /* stage('Lint') {
            steps {
                // Imprime un separador visual en los logs
                script {
                    echo '========== Running ESLint =========='
                }
                // Ejecuta ESLint para validar sintaxis y estilo de código
                sh '''
                    // Si hay errores de linting, el build falla aquí
                    bun run lint
                '''
            }
        } */

        // ====== STAGE 4: COMPILAR LA APLICACIÓN ======
        stage('Build') {
            steps {
                // Imprime un separador visual en los logs
                script {
                    echo '========== Building application =========='
                }
                // Compila TypeScript y genera el build optimizado con Vite
                sh '''
                    // Compila tipos TypeScript y crea bundle de producción
                    bun run build
                '''
            }
        }
        
        // ====== STAGE 5: Desplegar compilado ======
        stage('Archive Artifacts') {
            steps {
                // Imprime un separador visual en los logs
                script {
                    echo '========== Deploy build =========='
                }
                sh '''
                    // Instala el paquete serve
                    bun install -g serve
                    // Despliega el build en el puerto 3000
                    serve -s dist -l 3000 &
                '''
                
            }
        }

    }

    // ====== POST-STEPS: ACCIONES DESPUÉS DEL PIPELINE ======
    post {
        // Se ejecuta SIEMPRE, incluso si hay errores
        always {
            // Imprime un separador visual en los logs
            script {
                echo '========== Pipeline Execution Completed =========='
            }
            // Limpia el workspace eliminando archivos temporales
            cleanWs()
        }

        // Se ejecuta SOLO si el build fue exitoso
        success {
            // Imprime un separador visual en los logs
            script {
                echo '========== Build Successful =========='
            }
            // Aquí se podrían agregar notificaciones (Slack, email, etc.)
        }

        // Se ejecuta SOLO si el build falló
        failure {
            // Imprime un separador visual en los logs
            script {
                echo '========== Build Failed =========='
            }
            // Aquí se podrían agregar notificaciones de error
        }
    }
}
