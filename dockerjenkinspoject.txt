pipeline {
    
    agent any
    
    tools {
    maven 'maven3.9.6'
    }

    
    stages{
        
        
        
        //Git CheckOut the Code
        stage('CheckOutCode'){
            steps{
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/petergillgithub/java-web-app-docker.git']])
            }
        }
        
        //Build the Package
        stage('Build') {
            steps{
                sh "mvn clean package"
                
            }
        }
        
        //Docker build Image
        
        stage('DockerbuildImage'){
            steps{
                sh "docker build -t petergillhmg/java-web-app-docker:11 ."
            }
        }
        
        //Docker Login and push the image to DockerHub
        stage('DockerLogin and Push the image'){
            steps{
            withCredentials([string(credentialsId: 'Dockersshkeycredentials', variable: 'DockerPassword')]) {
            sh "docker login -u petergillhmg -p ${DockerPassword}"
            }
            sh "docker push petergillhmg/java-web-app-docker:11"
            }
        }
        
        //Deploy the app to docker server.
        stage('Deploy the app to docker server'){
            steps{
            sshagent(['Docker_Server_SSH']) {
            sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.19.14 docker rm -f javawebappcontainer || true"
            sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.19.14 docker run -d -p 8080:8080 --name javawebappcontainer petergillhmg/java-web-app-docker:11"
            }
                
            }
        }
        
        
    }
}
