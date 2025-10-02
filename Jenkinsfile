#!/user/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
  [
    $class: 'GitSCMSource',
    remote: 'https://github.com/leonnelzanguim/jenkins-shared-library.git',
    credentialsId: 'github-credentials'
  ]
)

def gv

pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    environment {
        IMAGE_NAME = '10012975/demo-app:java-maven-1.0'
    }
    stages {
        stage('build app') {
            steps {
                script {
                    echo 'building application jar...'
                    buildJar()
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                     echo "Deploying application to EC2...."
                    def dockerCmd = "docker run -p 3080:3080 -d ${IMAGE_NAME}"
                    sshagent(['ec2-server-key']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@35.159.46.245 ${dockerCmd}"
                    }
                }
            }
        }

        }
    }

