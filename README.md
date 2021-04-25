# Jenkins Pipeline

## A simple pipeline setup to automate docker builds.

- It automates docker builds, image creation, image upload and container provisioning.

- A completely dockerized python based application hosted in github is used for this.

# Pre-requistes:

1. Jenkins is up and running
2. Docker installed on Jenkins instance and configured.
3. Docker plug-in installed in Jenkins
4. User account setup in dockerhub
5. Port 8096 is opened up in firewall rules.

# Pipeline Syntax

```
pipeline{
    
    agent any
    environment {
        dockerImage = ''
        registry = 'shonevarkey/mypythonapp'
        registryCredential = 'dockerhub_id'
    }
    stages{
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/shonevarkey/mypythonrepo']]])
            }
            
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
                
            }
            
        }
        
        stage('Uploading Image') {
            steps {
                script {
                         docker.withRegistry( '', registryCredential ) {
                         dockerImage.push()
                     
                    }
                }
            }
        }
        
     // Stopping Docker containers for cleaner Docker run
     stage('docker stop container') {
         steps {
            sh 'docker ps -f name=mypythonappContainer -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=mypythonappContainer -q | xargs -r docker container rm'
         }
       }
       
       
     // Running Docker container, make sure port 8096 is opened in 
    stage('Docker Run') {
     steps{
         script {
            dockerImage.run("-p 8096:5000 --rm --name mypythonappContainer")
         }
      }
    }
        
    }
}

```



