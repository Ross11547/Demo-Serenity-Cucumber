pipeline {
    agent any

    tools {
        // el nombre debe ser EXACTAMENTE igual al de Jenkins
        maven 'Maven 3.8.6'
    }

    stages {
        stage('Checkout') {
            steps {
                //git 'https://github.com/Laura4lilavati/Demo-Serenity-Cucumber.git'
                // o tu fork:
                git 'https://github.com/Ross11547/Demo-Serenity-Cucumber.git'
            }
        }

        stage('Build & Test') {
            steps {
                bat 'mvn -version'
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
