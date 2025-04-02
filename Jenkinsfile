pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/AhnHao/B2111795_Tran_Anh_Hao_CT449_Frontend.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t vue-app:latest .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker stop vue-app || true && docker rm vue-app || true'
                sh 'docker run -d --name vue-app -p 80:80 vue-app:latest'
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
