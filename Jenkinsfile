#!groovy​

pipeline {
  agent {
    kubernetes {
        label 'pod'
        defaultContainer 'jnlp'
        yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    label: k8-pods
spec:
  containers:
  - name: "docker-builder"
    image: "docker:19"
    tty: true
    alwaysPullImage: true
    volumeMounts:
      - name: "docker-socket"
        mountPath: "/var/run/docker.sock"
  volumes:
    - name: "docker-socket"
      hostPath:
        path: "/var/run/docker.sock"
"""
    }
  }
  environment {
    ACR_SERVER = 'prudentialray.azurecr.io'
  }
  stages {
    stage('Deploy from master') {
      stages {
        stage ('build image') {
          steps {
            container('docker-builder') {
              dir('base') {
                withCredentials([usernamePassword(credentialsId: azure-credential, usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASSWORD')]{
                  sh 'docker login -u $ACR_USER -p $ACR_PASSWORD https://$env.ACR_SERVER'
                  // build image
                  def imageWithTag = "$env.ACR_SERVER/generic-base:$env.BUILD_NUMBER"
                  def image = docker.build imageWithTag
                  // push image
                  image.push()
                }
              }
            }
          }
        }
        stage('Terraform') {
          steps {
            container('deployer') {
              dir('terraform') {
                script {
                  echo "hello world"
                }
              }
            }
          }
        }
      }
    }
  }
}
