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
    DOCKER_REGISTRY_SERVER   = 'docker-rtsre-local.pruregistry.intranet.asia:8443'
    DOCKER_USERNAME          = "SRVMYRHOCICD@prudential.com.my"
    DOCKER_PASSWORD          = credentials('ARTI_SRVMYRHOCICD_RTSRE')
    HTTP_PROXY               = 'http://10.163.39.77:8080'
    HTTPS_PROXY              = 'http://10.163.39.77:8080'
    NO_PROXY                 = 'intranet.asia,pru.intranet.asia'
    TERRAFORM_MINOR_VERSIONS = '0.11 0.12 0.13'
    ANSIBLE_VERSION          = '2.7.5 2.9.0 2.10.0'
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
                sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD https://$DOCKER_REGISTRY_SERVER'
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
        stage ('Terraform image') {
          steps {
            container('docker-builder') {
              dir('terraform') {
                script {
                  def imageWithTag = "$DOCKER_REGISTRY_SERVER/jenkins-agent-terraform:${env.GIT_TAG}"
                  def image = docker.build(imageWithTag, "--build-arg TERRAFORM_MINOR_VERSIONS=\"$TERRAFORM_MINOR_VERSIONS\" --build-arg http_proxy=$HTTP_PROXY --build-arg https_proxy=$HTTPS_PROXY --build-arg no_proxy=$NO_PROXY .")
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
                  def imageWithTag = "$DOCKER_REGISTRY_SERVER/jenkins-agent-pwsh:${env.GIT_TAG}"
                  def image = docker.build(imageWithTag, "--build-arg http_proxy=$HTTP_PROXY --build-arg https_proxy=$HTTPS_PROXY --build-arg no_proxy=$NO_PROXY .")
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
