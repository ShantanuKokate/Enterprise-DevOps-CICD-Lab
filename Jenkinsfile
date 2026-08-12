pipeline {

    agent any

    environment {
        APP_NAME   = 'employee-management-system'
        IMAGE_NAME = 'employee-management-system:dev'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'pnpm install --frozen-lockfile'
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                sh 'mkdir -p dependency-check-report'

                withCredentials([
                    string(
                        credentialsId: 'nvd-api-key',
                        variable: 'NVD_API_KEY'
                    )
                ]) {
                    dependencyCheck(
                        odcInstallation: 'OWASP-Dependency-Check',
                        additionalArguments: '--project "Employee Management System" --scan . --format HTML --format XML --out dependency-check-report --nvdApiKey ' + env.NVD_API_KEY
                    )
                }
            }
        }

        stage('Publish OWASP Report') {
            steps {
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'dependency-check-report',
                    reportFiles: 'dependency-check-report.html',
                    reportName: 'OWASP Dependency-Check Report'
                ])
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
                    docker rm -f ${APP_NAME}-test 2>/dev/null || true

                    docker run -d \
                        --name ${APP_NAME}-test \
                        -p 8081:80 \
                        ${IMAGE_NAME}

                    sleep 5

                    curl -f http://localhost:8081

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