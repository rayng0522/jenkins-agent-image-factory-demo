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
  - name: "git"
    image: "alpine/git"
    command:
    - cat
    tty: true
    alwaysPullImage: true
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
        stage('Git Tag') {
          steps {
            container('git') {
              script {
                def gitTag = sh(returnStdout: true, script: "git tag --sort version:refname | tail -1").trim()
                env.GIT_TAG = gitTag
                echo "${env.GIT_TAG}"
              }
            }
          }
        }
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
        stage('Ansible image') {
          steps {
            container('docker-builder') {
              dir('ansible') {
                withCredentials([usernamePassword(credentialsId: 'azure-credential', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASSWORD')]) {
                  script {
                    sh 'docker login -u $ACR_USER -p $ACR_PASSWORD https://$ACR_SERVER'
                    def imageWithTag = "$ACR_SERVER/ansible:${env.GIT_TAG}"
                    def image = docker.build imageWithTag
                    image.push()
                  }
                }
              }
            }
          }
        }
        stage('Helper-script image') {
          steps {
            container('docker-builder') {
              dir('ansible') {
                withCredentials([usernamePassword(credentialsId: 'azure-credential', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASSWORD')]) {
                  script {
                    sh 'docker login -u $ACR_USER -p $ACR_PASSWORD https://$ACR_SERVER'
                    def imageWithTag = "$ACR_SERVER/helper_script:${env.GIT_TAG}"
                    def image = docker.build imageWithTag
                    image.push()
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
