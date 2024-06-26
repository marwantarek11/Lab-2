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
