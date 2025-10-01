def gv
pipeline {
  agent any
  tools{
    maven "maven-3.9"
  }
  stages {
    stage("init") {
      steps {
        script {
          gv = load "java-maven-app/script.groovy"
        }
      }
    }
    stage('increment version'){
      steps{
        script{
          echo 'increment app version...'
          sh 'mvn -f java-maven-app/pom.xml build-helper:parse-version versions:set -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} versions:commit'
          def matcher = readFile('java-maven-app/pom.xml') =~ '<version>(.+)</version>'
          def version = matcher[0][1]
          env.IMAGE_NAME = "$version-$BUILD_NUMBER"
        }
      }
    }
     stage("build Jar") {
      steps {
        script {
          gv.buildJar()
        }
      }
    }
    stage("build Image") {
      steps {
        script {
          gv.buildImage()
        }
      }
    }
    stage("deploy") {
      steps { 
        script {
          gv.deployApp()   
        }
      }
    }
    stage('commit version update'){
      steps{
        script{
          withCredentials([usernamePassword(credentialsId: 'github-credentials-push', passwordVariable: 'PASS', usernameVariable: 'USER')]){

            sh 'git config --global user.email "jenkins@example.com"'
            sh 'git config --global user.name "jenkins"'

            sh 'git remote -v'
            sh 'git status'
            sh 'git branch'
            sh 'git config --list'
            
            sh "git remote set-url origin https://${USER}:${PASS}@github.com/leonnelzanguim/build-automation-with-jenkins.git"
            sh 'git add .'
            sh 'git commit -m "ci: version bump"'
            sh 'git push origin HEAD:jenkins-jobs'
          }
        }
      }
    }
  }
}
