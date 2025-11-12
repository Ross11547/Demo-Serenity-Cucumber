pipeline {
  agent any
  options { skipDefaultCheckout(true) }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prep Maven (robusto)') {
      steps {
        bat '''
        setlocal EnableDelayedExpansion
        set MVER=3.9.9
        set ZIP=apache-maven-%MVER%-bin.zip
        set DIR=apache-maven-%MVER%

        if not exist .maven (
          echo ==== Descargando Maven %MVER% ====
          powershell -NoLogo -NoProfile -ExecutionPolicy Bypass -Command ^
            "$urls=@( ^
              'https://dlcdn.apache.org/maven/maven-3/%env:MVER%/binaries/%env:ZIP%', ^
              'https://archive.apache.org/dist/maven/maven-3/%env:MVER%/binaries/%env:ZIP%' ^
            ); ^
            $ok=$false; ^
            foreach($u in $urls){ try{ Write-Host ('Intentando ' + $u); Invoke-WebRequest -UseBasicParsing -Uri $u -OutFile 'maven.zip'; if((Get-Item 'maven.zip').Length -gt 0){ $ok=$true; break } } catch { } }; ^
            if(-not $ok){ Write-Error 'No se pudo descargar Maven desde ningun mirror.'; exit 1 }"

          powershell -NoLogo -NoProfile -ExecutionPolicy Bypass -Command ^
            "Expand-Archive -Path 'maven.zip' -DestinationPath . -Force"
          if not exist "%DIR%" (
            echo ERROR: No se descomprimio %DIR%.
            dir
            exit /b 1
          )
          ren "%DIR%" .maven
          del maven.zip
        )

        if not exist ".maven\\bin\\mvn.cmd" (
          echo ERROR: Maven no quedo disponible en .maven\\bin\\mvn.cmd
          exit /b 1
        )
        endlocal
        '''
      }
    }

    stage('Build & Test') {
      steps {
        bat '''
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
      archiveArtifacts artifacts: 'target/site/serenity/**', onlyIfSuccessful: false
    }
  }
}
