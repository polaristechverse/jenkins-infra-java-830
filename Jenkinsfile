pipeline {
    agent {
        label 'Dev'
    }
    environment {
        Image_Name = 'javaslim'
        Image_Version    = "${env.BUILD_NUMBER}"
    }
    stages{
        stage('check softwate'){
            steps {
                sh 'mvn -version'
                sh 'terraform version'
                sh 'packer version'
                sh 'aws --version'
                sh 'docker ps'
            }
        }
        stage('Maven Build'){
            steps{
                sh 'mvn -version'
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Docker Build'){
            steps{
                sh 'docker ps'
                sh 'docker build -t ${Image_Name}:v${Image_Version}'
            }
        }
        
    }
}