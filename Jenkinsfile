pipeline {
    agent {
        label 'Dev'
    }
    environment {
        Image_Name = 'javaslim'
        Image_Version    = "${env.BUILD_NUMBER}"
        DOCKER_CREDS = "DockerHubAccess"
        TF_DIR = "/home/ubuntu/workspace/infrapipeline_master"
    }
    parameters {
    choice(name: 'Docker_Build', choices: ['yes', 'no'], description: 'Build & Push Docker Image')
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
                sh 'docker build -t ${Image_Name}:v${Image_Version} .'
            }
        }
        stage('Docker Login') {
      when {
         expression { params.Docker_Build == 'yes' }
      }
      steps {
        withCredentials([usernamePassword(
          credentialsId: "${DOCKER_CREDS}",
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
        }
      }
    }
        stage('Docker Push'){
             when {
            expression { params.Docker_Build == 'yes' }
            }
            steps{
                sh "docker push $Image_Name:v$Image_Version"
            }
        }
        stage('Check Infra (Terraform State)') {
            steps {
              script {
                dir("${TF_DIR}") {

                  def status = sh(script: "terraform state list", returnStatus: true)

                  if (!status) {
                    echo "Infra not found → triggering infra pipeline..."

                    build job: 'infrapipeline/master',
                          wait: true,
                          parameters: [
                            string(name: 'REGION', value: 'ap-south-2'),
                            string(name: 'TERRAFORM_APPLY', value: 'yes'),
                            string(name: 'Ansible_Build', value: 'yes'),
                            string(name: 'Pull_AMI', value: 'yes')
                          ]

                    echo "Infra pipeline completed"
                  } else {
                    echo "Infra already exists (Terraform state present)"
                  }
                }
              }
            }
        }
        stage('k8s deployment'){
            agent {
                label 'k8smaster'
            }
            steps{
                sh "kubectl create deployment javaapp --image $Image_Name:v$Image_Version --replicas 3"
                sh "kubectl expose deployment javaapp --port 5600 --target 5600 --type NodePort"
            }
        }
        
    }
}