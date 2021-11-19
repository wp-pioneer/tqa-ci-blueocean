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
  pushd \\Games\\RunGame_Main && RunGame_Main_Test.bat
  exit /b 0
  """
  echo 'step: login'
  bat "pushd \\Auto && AutoHotkey.exe example.ahk ${name}"
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
              doWork('TQA_HIGHTEST1')
            } 
          }
        }

        stage('run2') {
          agent {
            label 'HIGHTEST2'
          }
          steps {
            script {
              doWork('TQA_HIGHTEST2')
            } 
          }
        }

        stage('run3') {
          agent {
            label 'HIGHTEST3'
          }
          steps {
            script {
              doWork('TQA_HIGHTEST3')
            } 
          }
        }

        stage('run4') {
          agent {
            label 'HIGHTEST4'
          }
          steps {
            script {
              doWork('TQA_HIGHTEST4')
            } 
          }
        }

        stage('run5') {
          agent {
            label 'LOWTEST1'
          }
          steps {
            script {
              doWork('TQA_LOWTEST1')
            } 
          }
        }

        stage('run6') {
          agent {
            label 'LOWTEST2'
          }
          steps {
            script {
              doWork('TQA_LOWTEST3')
            } 
          }
        }

        stage('run7') {
          agent {
            label 'LOWTEST4'
          }
          steps {
            script {
              doWork('TQA_LOWTEST4')
            } 
          }
        }

        stage('run8') {
          agent {
            label 'LOWTEST5'
          }
          steps {
            script {
              doWork('TQA_LOWTEST5')
            } 
          }
        }

        stage('run9') {
          agent {
            label 'MIDDLETEST1'
          }
          steps {
            script {
              doWork('TQA_MIDDLETEST1')
            } 
          }
        }

        stage('run10') {
          agent {
            label 'MIDDLETEST2'
          }
          steps {
            script {
              doWork('TQA_MIDDLETEST2')
            } 
          }
        }

        stage('run11') {
          agent {
            label 'MIDDLETEST3'
          }
          steps {
            script {
              doWork('TQA_MIDDLETEST3')
            } 
          }
        }

        stage('run12') {
          agent {
            label 'MIDDLETEST4'
          }
          steps {
            script {
              doWork('TQA_MIDDLETEST4')
            } 
          }
        }




      }
    }

  }
}