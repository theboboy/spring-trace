pipeline {
  agent any
  stages {
    stage('discard changes') {
      steps {
        sh 'git checkout .'
      }
    }
    stage('pull latest code') {
      steps {
        sh 'git pull'
      }
    }
    stage('build') {
      steps {
        sh 'mvn clean compile'
      }
    }
  }
}