pipeline {
    agent any

    environment {
        IMAGE_NAME = "aa3000/iportfolio"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/alok-singh1223/iPortfolio-1.1.1.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                        echo %DOCKER_PASS% | docker login -u "%DOCKER_USER%" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {

            steps {

                bat 'docker push %IMAGE_NAME%:%IMAGE_TAG%'

            }
        }
        stage('Deploy to Kubernetes') {

             steps {
                 bat '''
                 set "KUBECONFIG=C:\\Users\\ALOK SINGH\\.kube\\config"

                 kubectl apply -f deployment.yaml
                 if errorlevel 1 exit /b 1
                
                 kubectl apply -f service.yaml
                 if errorlevel 1 exit /b 1

                 kubectl set image deployment/iportfolio iportfolio=%IMAGE_NAME%:%IMAGE_TAG%
                 if errorlevel 1 exit /b 1

                 kubectl rollout status deployment/iportfolio --timeout=180s
                 if errorlevel 1 exit /b 1
                 '''
             }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the Jenkins Console Output.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
