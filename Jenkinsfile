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
  imagePullSecrets:
    - name: "aws-ecr-secret"
  containers:
  - name: deployer
    image: 578612111946.dkr.ecr.ap-southeast-2.amazonaws.com/helm-aks-deployer:latest
    command:
    - cat
    tty: true
    alwaysPullImage: true
  - name: "docker-builder"
    image: "docker:18"
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
  stages {
    stage('Deploy from master') {
      stages {
        stage ('build image') {
          steps {
            container('docker-builder') {
              dir('test-sample-crud-api') {
                sh """
                  apk add python py-pip
                 """
                script {
                  echo "hello world"
                }
              }
            }
          }
        }
        stage('Terraform') {
          steps {
            container('deployer') {
              dir('terraform') {
                sh """
                  ls
                """
              }
            }
          }
        }
      }
    }
  }
}
