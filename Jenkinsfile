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
    ACR_SERVER      = 'prudentialray.azurecr.io'
    ANSIBLE_VERSION = '2.10.0'
    AZURE_CLIENT    = credentials('azure-credential')
    TERRAFORM_MINOR_VERSIONS = '0.11 0.12 0.13'
  }
  stages {
    stage('Jenkins agent container factory') {
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
        stage('Docker login') {
          steps {
            container('docker-builder') {
              script {
                sh 'docker login -u $AZURE_CLIENT_USR -p $AZURE_CLIENT_PSW https://$ACR_SERVER'
              }
            }
          }
        }
        stage ('Generic image') {
          steps {
            container('docker-builder') {
              dir('base') {
                script {
                  def imageWithTag = "$ACR_SERVER/generic-base:latest"
                  def image = docker.build imageWithTag
                  image.push()
                }
              }
            }
          }
        }
        stage ('Terraform image') {
          steps {
            container('docker-builder') {
              dir('terraform') {
                script {
                  def imageWithTag = "$ACR_SERVER/terraform:${env.GIT_TAG}"
                  def image = docker.build imageWithTag
                  image.push()
                }
              }
            }
          }
        }
        stage('Ansible image') {
          steps {
            container('docker-builder') {
              dir('ansible') {
                script {
                  def imageWithTag = "$ACR_SERVER/ansible:$ANSIBLE_VERSION"
                  def image = docker.build(imageWithTag, "--build-arg ANSIBLE_VERSION=$ANSIBLE_VERSION .")
                  image.push()
                }
              }
            }
          }
        }
        stage('Helper-script image') {
          steps {
            container('docker-builder') {
              dir('helper-script') {
                script {
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
