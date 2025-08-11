pipeline {
  agent any
  options { timestamps() }

  // If you do NOT use a GitHub webhook, uncomment the next line to poll periodically:
  // triggers { pollSCM('H/10 * * * *') } // every ~10 min

  stages {
    stage('Checkout') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[
            url: 'https://github.com/srinivassivakumar/testingsri.git',
            credentialsId: 'github'   // <-- use your actual Jenkins credential ID
          ]]
        ])
      }
    }

    stage('Setup Python venv') {
      steps {
        bat '''
        @echo on
        cd /d "%WORKSPACE%"
        where py || where python
        if %ERRORLEVEL% NEQ 0 (
          echo [ERROR] Python not found in PATH for the Jenkins service user.
          echo Install Python 3 or set an absolute path (e.g., C:\\Python311\\python.exe)
          exit /b 1
        )

        rem Prefer the py launcher if present
        py -3 --version >nul 2>&1 && (set "PY=py -3") || (set "PY=python")

        %PY% -m venv .venv
        call .venv\\Scripts\\activate.bat
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
        python practise.py > run.log 2>&1
        type run.log
        '''
      }
    }

    stage('Publish HTML (optional)') {
      when { expression { fileExists('code.html') } }
      steps {
        // Requires "HTML Publisher" plugin (Manage Plugins)
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
