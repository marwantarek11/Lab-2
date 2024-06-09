# Python-App Deployment on OpenShift using Jenkins Pipeline

## Project Overview

This project involves cloning an Python application, creating a Dockerfile for the application, pushing the Dockerfile to the repository, and setting up a CI/CD pipeline using Jenkins to deploy the application to OpenShift. The pipeline will be configured to build a Docker image, push it to Docker Hub, edit the deployment configuration, and deploy the application to OpenShift. Additionally, Jenkins shared libraries will be utilized, and pipeline post actions will be set for different build outcomes.

## Prerequisites

Before starting, ensure you have the following:

1. **Git**: For cloning repositories and pushing changes.
2. **Docker**: For building Docker images.
3. **Jenkins**: Installed and configured.
4. **OpenShift**: Access to an OpenShift environment.
5. **Docker Hub Account**: For pushing Docker images.
6. **Jenkins Shared Libraries**: Set up in your Jenkins instance.

## Pipeline Stages
- Buid Docker image from Dockerfile in GitHub
- Push image to Docker hub
- Edit new image in deployment.yaml file
- Deploy Image On Openshift

## Steps

### 1. Clone the Application Repository

Clone the repository from GitHub:

```bash
git clone https://github.com/IbrahimAdell/Lab.git
cd Lab
```
Push the files to your repo

### 2. Set Up GitHub Shared Library [marwantarek11/Shared-Library-lab](https://github.com/marwantarek11/shared-library-lab.oit.git)

1- buildDockerImage.groovy
```groovy
#!usr/bin/env groovy
def call(String imageName) {

        // Build and push Docker image
        echo "Building Docker image..."
        sh "docker build -t ${imageName}:${BUILD_NUMBER} ."
 
}
```
2- pushDockerImage.groovy
```groovy
#!usr/bin/env groovy
def call(String dockerHubCredentialsID, String imageName) {

	// Log in to DockerHub 
	withCredentials([usernamePassword(credentialsId: "${dockerHubCredentialsID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
		sh "docker login -u ${USERNAME} -p ${PASSWORD}"
        }
        
        // Build and push Docker image
        echo "Pushing Docker image..."
        sh "docker push ${imageName}:${BUILD_NUMBER}"	 
}
```
3- editDeploymentYaml.groovy
```groovy
#!/usr/bin/env groovy

// KubernetesCredentialsID 'KubeConfig file'
def call(String imageName) {
    
    // Edit deployment.yaml with new Docker Hub image
    sh "sed -i 's|image:.*|image: ${imageName}:${BUILD_NUMBER}|g' deployment.yml"

}
```
4- deployTooc.groovy
```groovy
#!/usr/bin/env groovy

def call(String openshiftCredentialsID, String nameSpace, String clusterUrl) {

    
    // Login to OpenShift using the service account token
    withCredentials([string(credentialsId: openshiftCredentialsID, variable: 'OC_TOKEN')]) {
        sh "oc login --token=$OC_TOKEN --server=$clusterUrl --insecure-skip-tls-verify"
    }

    // Apply the updated deployment.yaml to the OpenShift cluster
    sh "oc apply -f . --namespace=${nameSpace}"
}
```


### 3. Configure Jenkins System
![1](https://github.com/marwantarek11/Lab-2/assets/167176241/2b68fba0-eb79-4ab6-a879-7d8e54235872)
![2](https://github.com/marwantarek11/Lab-2/assets/167176241/9ca1b1ac-7a2a-48be-8f62-e07a91180293)


### 4. Configure Jenkins Credentials
Add credentials for accessing the OpenShift cluster. This can be done by creating a new OpenShift token credential in Jenkins credentials.

![3](https://github.com/marwantarek11/Lab-2/assets/167176241/d85d03d9-3873-4989-a8b8-16bbacab420f)


### 5. Configure Jenkins Pipeline
Set up a Jenkins pipeline job that fetches the code from the application repository, builds the Docker image, pushes it to the OpenShift registry, and deploys the application to the OpenShift cluster using the shared library.

![4](https://github.com/marwantarek11/Lab-2/assets/167176241/0f299fae-0e64-4e00-91c6-636ed6ced604)

```groovy
@Library('Jenkins-Shared-Library') _

pipeline {
    agent any
    environment {
        dockerHubCredentialsID = 'DockerHub'
        imageName = 'marwantarek11/oc-python-app'
        openshiftCredentialsID = 'openshift'
        nameSpace = 'marwantarek'
        clusterUrl = 'https://api.ocp-training.ivolve-test.com:6443'
    }
    stages {
        stage('Build Docker image from Dockerfile in GitHub') {
            steps {
                script {
                    buildDockerImage(imageName)
                }
            }
        }
        stage('Push image to Docker hub') {
            steps {
                script {
                    pushDockerImage(dockerHubCredentialsID, imageName)
                }
            }
        }
        stage('Edit new image in deployment.yml file') {
            steps {
                script {
                    dir('oc') {
                        editDeploymentYaml(imageName)
                    }
                }
            }
        }
        stage('Deploy on OpenShift Cluster') {
            steps {
                script {
                    dir('oc') {
                        deployTooc(openshiftCredentialsID, nameSpace, clusterUrl)
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                echo 'This runs always'
            }
        }
        success {
            script {
                echo 'This runs on success'
            }
        }
        failure {
            script {
                echo 'This runs on failure'
            }
        }
    }
}
```

### 5. Jenkins Pipeline Stages
![5](https://github.com/marwantarek11/Lab-2/assets/167176241/f1788969-8d0d-495b-87f3-fab37a161ae5)
![6](https://github.com/marwantarek11/Lab-2/assets/167176241/5b3c818c-092a-4732-830b-fe2b6ae8ae6b)
![7](https://github.com/marwantarek11/Lab-2/assets/167176241/02e89957-cffe-4d5a-bbf6-47a8d4a756a9)
