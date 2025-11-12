pipeline {
  agent any
  options { skipDefaultCheckout(true) }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build & Test') {
      steps {
        bat '''
        rem ==== Descargar Maven local (una vez) ====
        set MVER=3.9.9
        if not exist .maven (
          powershell -NoProfile -ExecutionPolicy Bypass ^
            "Invoke-WebRequest https://downloads.apache.org/maven/maven-3/%MVER%/binaries/apache-maven-%MVER%-bin.zip -OutFile maven.zip"
          powershell -NoProfile -ExecutionPolicy Bypass ^
            "Expand-Archive maven.zip -DestinationPath . -Force"
          ren apache-maven-%MVER% .maven
          del maven.zip
        )

        rem ==== Verifica Java y ejecuta con Maven local ====
        where java || (echo ERROR: Java no esta en PATH.& exit /b 1)
        .\\.maven\\bin\\mvn.cmd -v
        .\\.maven\\bin\\mvn.cmd -B -U clean verify
        '''
      }
    }

    stage('Report') {
      steps {
        publishHTML(target: [
          reportDir: 'target/site/serenity',
          reportFiles: 'index.html',
          reportName: 'Serenity Report',
          keepAll: true, alwaysLinkToLastBuild: true, allowMissing: false
        ])
      }
    }
  }

  post {
    always {
      junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml,target/failsafe-reports/*.xml'
      archiveArtifacts artifacts: 'target/site/serenity/**', onlyIfSuccessful: false
    }
  }
}
