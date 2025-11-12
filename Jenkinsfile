pipeline {
  agent any
  options { skipDefaultCheckout(true) }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prep Maven (sin PowerShell)') {
      steps {
        bat '''
        setlocal
        set MVER=3.9.9
        if not exist .maven (
          echo ==== Descargando Maven %MVER% ====
          for %%U in (
            https://dlcdn.apache.org/maven/maven-3/%MVER%/binaries/apache-maven-%MVER%-bin.zip
            https://archive.apache.org/dist/maven/maven-3/%MVER%/binaries/apache-maven-%MVER%-bin.zip
          ) do (
            echo Intentando %%U
            curl.exe -L --retry 3 --retry-delay 2 -o maven.zip "%%U"
            if exist maven.zip goto :gotzip
          )
          echo ERROR: No se pudo descargar Maven desde ningun mirror.
          exit /b 1

          :gotzip
          echo ==== Extrayendo Maven ====
          powershell -NoLogo -NoProfile -ExecutionPolicy Bypass -Command "Expand-Archive -Path 'maven.zip' -DestinationPath '.' -Force"
          if not exist "apache-maven-%MVER%" (
            echo ERROR: No se encontro carpeta apache-maven-%MVER% tras descomprimir.
            dir
            exit /b 1
          )
          ren "apache-maven-%MVER%" .maven
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
