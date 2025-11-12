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
        @echo off
        setlocal ENABLEDELAYEDEXPANSION
        set MVER=3.9.9
        set ZIP=apache-maven-%MVER%-bin.zip
        set DIR=apache-maven-%MVER%

        if exist .maven\\bin\\mvn.cmd goto done

        echo ==== Descargando Maven %MVER% ====
        set URL1=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/%MVER%/%ZIP%
        set URL2=https://archive.apache.org/dist/maven/maven-3/%MVER%/binaries/%ZIP%
        set URL3=https://dlcdn.apache.org/maven/maven-3/%MVER%/binaries/%ZIP%

        if exist maven.zip del /q maven.zip

        call :trydl "%%URL1%%"
        if not exist maven.zip call :trydl "%%URL2%%"
        if not exist maven.zip call :trydl "%%URL3%%"

        if not exist maven.zip (
          echo ERROR: No se pudo descargar Maven desde ningun mirror.
          exit /b 1
        )

        for %%A in (maven.zip) do set SIZE=%%~zA
        if not defined SIZE (
          echo ERROR: Descarga vacia.
          exit /b 1
        )
        if !SIZE! LSS 1000000 (
          echo ERROR: Zip corrupto (solo !SIZE! bytes).
          del /q maven.zip
          exit /b 1
        )

        echo ==== Extrayendo Maven (tar primero, luego PowerShell si falla) ====
        tar -xf maven.zip 2>nul
        if not exist "%DIR%" (
          powershell -NoLogo -NoProfile -ExecutionPolicy Bypass -Command "Expand-Archive -Path 'maven.zip' -DestinationPath '.' -Force"
        )
        if not exist "%DIR%" (
          echo ERROR: No se pudo extraer %DIR%.
          dir
          exit /b 1
        )

        ren "%DIR%" .maven
        del /q maven.zip

        :done
        if not exist ".maven\\bin\\mvn.cmd" (
          echo ERROR: Maven no quedo en .maven\\bin\\mvn.cmd
          exit /b 1
        )
        exit /b 0

        :trydl
        set URL=%~1
        echo Intentando %URL%
        curl.exe -fSL -o maven.zip %URL% 2>nul
        exit /b 0
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
