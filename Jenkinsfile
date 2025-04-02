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

        stage('Stop & Remove Old Container') {
            steps {
                sh 'docker stop vue-app || true && docker rm vue-app || true'
            }
        }

        stage('Run New Container') {
            steps {
                sh 'docker run -d --restart always --name vue-app -p 80:80 vue-app:latest'
            }
        }
    }
}
