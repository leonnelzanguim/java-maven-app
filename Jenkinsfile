pipeline {   
    agent any
    stages {
        stage("test") {
            steps {
                script {
                    echo "Testing the application...."
                }
            }
        }
        
        stage("build") {
            steps {
                script {
                    echo "Building the application...."
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    echo "Deploying the application...."
                    def dockerCmd = 'docker run -p 3080:3080 -d 10012975/demo-app:1.0'
                    sshagent(['ec2-server-key']) {
                        ssh "ssh -o StrictHostKeyChecking=no ec2-user@35.159.46.245 ${dockerCmd}"
                    }
                    
                }
            }
        }               
    }
} 
