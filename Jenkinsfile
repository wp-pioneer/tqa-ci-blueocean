def doWork( final String name) {
  echo 'startingg...'
  echo 'syncing macro script...'
  bat "
  setlocal EnableDelayedExpansion
  robocopy \\kate.oscarmike.io\SharedDDC\Auto c:\Auto /MIR /s /TEE
  exit /b 0
  " 
  echo 'start main script'
  bat """
  setlocal EnableDelayedExpansion
  taskkill /f /im BravoHotel*
  taskkill /f /im AutoHotKey*
  pushd \\Games\\RunGame_Main && RunGame_Main_Test.bat
  exit /b 0
  """
  echo 'step: login'
  bat 'pushd \\Auto && AutoHotkey.exe example.ahk'
  echo 'step: upload log'
  bat 'pushd \\Auto && py pakinfo_upload.py'
}


pipeline {
  agent none
  stages {
    stage('parallels stage') {
      parallel {
      
        stage('run1') {
          agent {
            label 'HIGHTEST1'
          }
          steps {
            script {
              doWork('HIGHTEST1')
            } 
          }
        }

        stage('run2') {
          agent {
            label 'HIGHTEST2'
          }
          steps {
            script {
              doWork('HIGHTEST2')
            } 
          }
        }

        stage('run3') {
          agent {
            label 'HIGHTEST3'
          }
          steps {
            script {
              doWork('HIGHTEST3')
            } 
          }
        }

        stage('run4') {
          agent {
            label 'HIGHTEST4'
          }
          steps {
            script {
              doWork('HIGHTEST4')
            } 
          }
        }




      }
    }

  }
}