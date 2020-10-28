#!groovy​
@Library('jenkins_libraries')_

pipeline {
  agent {
    kubernetes {
        label 'pod'
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
  - name: "az-cli"
    image: "microsoft/azure-cli"
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
    TERRAFORM_MINOR_VERSIONS = '0.11 0.12 0.13'
    ANSIBLE_VERSIONS         = '2.7.5 2.9.0 2.10.0'
    PS_VERSIONS              = '6.2.0 7.0.1'
  }
  stages {
    stage('Jenkins agent images factory') {
      stages {
        stage('Git Tag') {
          steps {
            container('git') {
              script {
                def gitTag         = sh(returnStdout: true, script: "git tag --sort version:refname | tail -1").trim()
                def committerEmail = sh(returnStdout: true, script: 'git --no-pager show -s --format=\'%ae\'').trim()
                env.GIT_TAG         = gitTag
                env.COMMITTER_EMAIL = committerEmail
                echo "tag version is ${env.GIT_TAG} and recipient email is ${env.COMMITTER_EMAIL}"
              }
            }
          }
        }
        stage('Docker login') {
          steps {
            container('docker-builder') {
              script {
                echo "hello"
              }
            }
          }
        }
        stage ('Generic image') {
          steps {
            container('docker-builder') {
              dir('base') {
                script {
                  echo "hello"
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
                  echo "hello"
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
                  echo "hello"
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
                  echo "hello"
                }
              }
            }
          }
        }
        stage('Archive log to Azure Blob') {
          steps {
            container('az-cli') {
                archive_log("jenkinsarchivelogs")
            }
          }
        }
      }
    }
  }
  post {
    success {
      custom_email_notification("SUCCESSFUL", ["${env.COMMITTER_EMAIL}"])
      email_notification("SUCCESSFUL", ["${env.COMMITTER_EMAIL}"])
    }
    failure {
      custom_email_notification("SUCCESSFUL", ["${env.COMMITTER_EMAIL}"])
      email_notification("FAIL", ["${env.COMMITTER_EMAIL}"])
    }
  }
}
