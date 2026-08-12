pipeline {

    agent any

    environment {
        APP_NAME   = 'employee-management-system'
        IMAGE_NAME = 'employee-management-system:dev'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pnpm install --frozen-lockfile'
            }
        }

        stage('Build Application') {
            steps {
                sh 'pnpm build'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQube-Scanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=Employee-Management-System \
                            -Dsonar.projectName='Employee Management System' \
                            -Dsonar.sources=src
                        """
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Docker Test') {
            steps {
                sh '''
                    # Remove any previous test container safely
                    docker rm -f ${APP_NAME}-test 2>/dev/null || true

                    # Start the application container
                    docker run -d \
                        --name ${APP_NAME}-test \
                        -p 8081:80 \
                        ${IMAGE_NAME}

                    # Give Nginx time to start
                    sleep 5

                    # Test application
                    curl -f http://localhost:8081

                    # Cleanup test container
                    docker rm -f ${APP_NAME}-test
                '''
            }
        }
    }

    post {
        always {
            sh 'docker ps -a'
        }

        success {
            echo 'Dev/Test pipeline completed successfully.'
        }

        failure {
            echo 'Dev/Test pipeline failed.'
        }
    }
}