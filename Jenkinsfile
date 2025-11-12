pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/Ross11547/Demo-Serenity-Cucumber.git'
      }
    }
    stage('Build & Test') {
      steps {
        bat 'mvn clean verify'
      }
    }
    stage('Report') {
      steps {
        publishHTML([reportDir: 'target/site/serenity', reportFiles: 'index.html', reportName: 'Serenity Report'])
      }
    }
  }
}
