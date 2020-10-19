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
    stage('Build image') {
      stages {
        stage ('Generic image') {
          steps {
            container('docker-builder') {
              dir('base') {
                withCredentials([usernamePassword(credentialsId: 'azure-credential', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASSWORD')]) {
                  script {
                    sh 'docker login -u $ACR_USER -p $ACR_PASSWORD https://$ACR_SERVER'
                    def imageWithTag = "$ACR_SERVER/generic-base:latest"
                    def image = docker.build imageWithTag
                    image.push()
                  }
                }
              }
            }
          }
        }
        stage('Terraform image') {
          steps {
            container('deployer') {
              dir('terraform') {
                withCredentials([usernamePassword(credentialsId: 'azure-credential', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASSWORD')]) {
                  script {
                    sh 'docker login -u $ACR_USER -p $ACR_PASSWORD https://$ACR_SERVER'
                    def imageWithTag = "$ACR_SERVER/terraform:latest"
                    def image = docker.build imageWithTag
                    image.push()
                    def gitTag = sh(returnStdout: true, script: "git tag --sort version:refname | tail -1").trim()
                    echo "$gitTag"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
