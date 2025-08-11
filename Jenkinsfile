pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[
            url: 'https://github.com/srinivassivakumar/testingsri.git',
            credentialsId: 'github'   // <-- your credential ID
          ]]
        ])
      }
    }

    stage('Setup Python venv') {
      steps {
        bat '''
        @echo on
        cd /d "%WORKSPACE%"
        rem Create and activate venv using the Python launcher
        py -3 -m venv .venv
        if %ERRORLEVEL% NEQ 0 (
          echo [ERROR] Failed to create venv with py -3
          exit /b 1
        )
        call .venv\\Scripts\\activate.bat

        rem Upgrade pip and install deps if present
        python -m pip install --upgrade pip
        if exist requirements.txt (
          python -m pip install -r requirements.txt
        )
        '''
      }
    }

    stage('Run practise.py') {
      steps {
        bat '''
        @echo on
        cd /d "%WORKSPACE%"
        call .venv\\Scripts\\activate.bat
        python --version
        echo --- Running practise.py ---
        python practise.py > run.log 2>&1
        echo --- Script output (run.log) ---
        type run.log
        '''
      }
    }

    stage('Publish HTML (optional)') {
      when { expression { fileExists('code.html') } }
      steps {
        publishHTML(target: [
          reportDir: '.',
          reportFiles: 'code.html',
          reportName: 'Tic-Tac-Toe',
          keepAll: true,
          alwaysLinkToLastBuild: true
        ])
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'run.log, output/**, *.txt', allowEmptyArchive: true
    }
  }
}
