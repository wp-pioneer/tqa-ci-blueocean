def doWork( final String name) {
  echo 'startingg...'
  echo 'syncing macro script...'
  bat """
  setlocal EnableDelayedExpansion
  robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Auto c:\\Auto /MIR /s /TEE
  exit /b 0
  """
  echo 'start main script'
  bat """
  setlocal EnableDelayedExpansion
  taskkill /f /im BravoHotel*
  taskkill /f /im AutoHotKey*
  pushd \\Games\\RunGame_QA && RunGame_QA_Test.bat
  exit /b 0
  """
  echo 'step: login'
  bat "pushd \\Auto && AutoHotkey.exe example.ahk ${name}"
  //echo 'step: upload log'
  //bat 'pushd \\Auto && py pakinfo_upload.py'
}

def list = ["Test-1", "Test-2", "Test-3", "Test-4", "Test-5"]

pipeline {
  agent none
  triggers {
    cron('0 */1 * * *')
  }
  stages {
    stage('병렬처리') {
      parallel {
        steps {
          script {
            for(int i=0; i < list.size(); i++) {
                stage(list[i]){
                    echo "Element: $i"
                }
            }
          }
        }
      }
    }

  }
}