pipeline {
  agent any
  options { skipDefaultCheckout(true) }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Prep Maven') {
      steps {
        powershell '''
          $ErrorActionPreference = "Stop"
          $ver="3.9.9"; $zip="apache-maven-$ver-bin.zip"; $dir="apache-maven-$ver"
          if (-not (Test-Path ".maven\\bin\\mvn.cmd")) {
            $urls=@(
              "https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/$ver/$zip",
              "https://archive.apache.org/dist/maven/maven-3/$ver/binaries/$zip"
            )
            Remove-Item -Force -ErrorAction SilentlyContinue maven.zip
            foreach($u in $urls){
              try { Invoke-WebRequest -UseBasicParsing -Uri $u -OutFile maven.zip; if((Get-Item maven.zip).Length -gt 1MB){ break } }
              catch { }
            }
            if (-not (Test-Path maven.zip)) { throw "No se pudo descargar Maven." }
            Expand-Archive -Path maven.zip -DestinationPath . -Force
            if (Test-Path ".maven"){ Remove-Item -Recurse -Force ".maven" }
            Rename-Item $dir ".maven"
            Remove-Item maven.zip
          }
        '''
      }
    }

    stage('Build & Test') {
      steps {
        bat '''
        where java || (echo ERROR: Java no esta en PATH.& exit /b 1)
        .\\.maven\\bin\\mvn.cmd -v
        rem ====== COMPILAR Y EJECUTAR TESTS ======
        .\\.maven\\bin\\mvn.cmd -B -U clean verify
        '''
      }
    }

    stage('Report') {
      when { expression { return fileExists('target/site/serenity/index.html') } }
      steps {
        publishHTML(target: [
          reportDir: 'target/site/serenity',
          reportFiles: 'index.html',
          reportName: 'Serenity Report',
          keepAll: true,
          alwaysLinkToLastBuild: true,
          allowMissing: false
        ])
      }
    }
  }

  post {
    always {
      junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml,target/failsafe-reports/*.xml'
      archiveArtifacts artifacts: 'target/site/serenity/**', fingerprint: false, onlyIfSuccessful: false
      script {
        if (!fileExists('target/site/serenity/index.html')) {
          echo 'Serenity no generó HTML. Revisa que existan tests y que se ejecuten (mvn verify).'
        }
      }
    }
  }
}
