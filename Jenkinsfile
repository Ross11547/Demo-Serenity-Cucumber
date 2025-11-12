pipeline {
  agent any
  options { skipDefaultCheckout(true) }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prep Maven') {
      steps {
        powershell '''
          $ErrorActionPreference = "Stop"
          $ver = "3.9.9"
          $zip = "apache-maven-$ver-bin.zip"
          $dir = "apache-maven-$ver"

          if (-not (Test-Path ".maven\\bin\\mvn.cmd")) {
            Write-Host "==== Descargando Maven $ver ===="
            $urls = @(
              "https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/$ver/$zip",
              "https://archive.apache.org/dist/maven/maven-3/$ver/binaries/$zip"
            )

            Remove-Item -Force -ErrorAction SilentlyContinue maven.zip
            $downloaded = $false
            foreach ($u in $urls) {
              try {
                Write-Host "Intentando $u"
                Invoke-WebRequest -UseBasicParsing -Uri $u -OutFile "maven.zip"
                if ((Get-Item "maven.zip").Length -gt 1MB) { $downloaded = $true; break }
                else { Write-Warning "Zip muy pequeño, reintentando con otro mirror..." }
              } catch { Write-Warning $_.Exception.Message }
            }
            if (-not $downloaded) { throw "No se pudo descargar Maven desde ningun mirror." }

            Write-Host "==== Extrayendo Maven ===="
            Expand-Archive -Path "maven.zip" -DestinationPath "." -Force
            if (-not (Test-Path $dir)) { throw "No se encontró carpeta $dir tras descomprimir." }

            if (Test-Path ".maven") { Remove-Item -Recurse -Force ".maven" }
            Rename-Item $dir ".maven" -Force
            Remove-Item -Force maven.zip
          }

          if (-not (Test-Path ".maven\\bin\\mvn.cmd")) { throw "Maven no quedó disponible en .maven\\bin\\mvn.cmd" }
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
