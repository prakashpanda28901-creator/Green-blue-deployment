pipeline {
    agent any

    environment {
        IMAGE_NAME = "bluegreen-app"
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Build docker Image'){
            steps {
                sh "docker build -t $IMAGE_NAME ."
            }
        }
        stage("Run Tests") {
            steps {
                sh "python -m unittest"
            }
        }
        stage("Deploy Green Environment") {
            steps {
                sh '''
                docker stop green || true
                docker rm green || true
                
                docker run -d \
                --name green \
                -p 5001:5000
                bluegreen-app
                '''
            }
        }
        stage('Health Check') {
            steps {
                sh 'curl http://localhost:5001'
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh '''
                docker stop blue || true
                docker rm blue || true

                docker rename green blue
                '''
            }
        }
    }

    post {
        success {
            echo 'Blue-Green Deployment Successful'
        }

        failure {
            echo 'Deployment Failed'
        }
    }
}
