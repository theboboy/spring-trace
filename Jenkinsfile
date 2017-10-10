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
        sh 'git checkout origin/master'
        sh 'git pull'
      }
    }
    stage('clean project') {
      steps {
        echo 'mvn clean compile'
      }
    }
    stage('build project') {
      steps {
        echo 'mvn clean install -DskipTests'
      }
    }
  }
}