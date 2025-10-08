pipeline {
    agent any

    environment {
        DB_DIR = "C:/Jenkins/workspace/MongoDB"
        BACKEND_DIR = "C:/Jenkins/workspace/Backend"
        FRONTEND_DIR = "C:/Jenkins/workspace/Frontend"
    }

    stages {

        stage('Checkout MongoDB Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/rohanjob/Dataabase.git'
            }
        }

        stage('Setup MongoDB') {
            steps {
                echo "Starting MongoDB..."
                bat 'net start MongoDB'
                bat "mongo < ${DB_DIR}/init.js"
            }
        }

        stage('Checkout Backend Repo') {
            steps {
                dir("${BACKEND_DIR}") {
                    git branch: 'main', url: 'https://github.com/rohanjob/Backend.git'
                }
            }
        }

        stage('Checkout Frontend Repo') {
            steps {
                dir("${FRONTEND_DIR}") {
                    git branch: 'main', url: 'https://github.com/rohanjob/Frontend.git'
                }
            }
        }

        stage('Deploy Backend & Frontend in Parallel') {
            parallel {
                stage('Backend') {
                    steps {
                        dir("${BACKEND_DIR}") {
                            bat 'npm install'
                            bat 'node index.js'  // Start backend server
                        }
                    }
                }

                stage('Frontend') {
                    steps {
                        dir("${FRONTEND_DIR}") {
                            bat 'npm install'
                            bat 'npm run build'
                            bat 'npx serve -s build -l 3000'
                        }
                    }
                }
            }
        }

    }

    post {
        success { echo "Full-Stack Multi-Repo Deployment Successful!" }
        failure { echo "Deployment Failed!" }
    }
}
