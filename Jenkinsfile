pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Laura4lilavati/Demo-Serenity-Cucumber.git'
                // si quieres usar tu fork, cambia la URL a:
                // git 'https://github.com/Ross11547/Demo-Serenity-Cucumber.git'
            }
        }

        stage('Build & Test') {
            steps {
                bat 'mvn clean verify'
            }
        }

        stage('Report') {
            steps {
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'target/site/serenity',
                    reportFiles: 'index.html',
                    reportName: 'Serenity Report'
                ])
            }
        }
    }
}
