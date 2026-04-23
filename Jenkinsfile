pipeline {
    agent {
        label 'Dev'
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
    }
}