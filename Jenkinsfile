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


pipeline {
  agent none
  triggers {
    cron('0 19-9/1 * * *')
  }
  stages {
    stage('parallels stage') {
      parallel {
      
        stage('HIGHTEST1') {
          agent {
            label 'HIGHTEST1'
          }
          steps {
            script {
              doWork('TQA_HIGHTEST1')
            } 
          }
        }

        stage('HIGHTEST2') {
          agent {
            label 'HIGHTEST2'
          }
          steps {
            script {
              doWork('TQA_HIGHTEST2')
            } 
          }
        }

        stage('HIGHTEST3') {
          agent {
            label 'HIGHTEST3'
          }
          steps {
            script {
              doWork('TQA_HIGHTEST3')
            } 
          }
        }

        stage('HIGHTEST4') {
          agent {
            label 'HIGHTEST4'
          }
          steps {
            script {
              doWork('TQA_HIGHTEST4')
            } 
          }
        }

        stage('LOWTEST1') {
          agent {
            label 'LOWTEST1'
          }
          steps {
            script {
              doWork('TQA_LOWTEST1')
            } 
          }
        }

        stage('LOWTEST2') {
          agent {
            label 'LOWTEST2'
          }
          steps {
            script {
              doWork('TQA_LOWTEST2')
            } 
          }
        }

        stage('LOWTEST3') {
          agent {
            label 'LOWTEST3'
          }
          steps {
            script {
              doWork('TQA_LOWTEST3')
            } 
          }
        }

        stage('LOWTEST4') {
          agent {
            label 'LOWTEST4'
          }
          steps {
            script {
              doWork('TQA_LOWTEST4')
            } 
          }
        }

        stage('LOWTEST5') {
          agent {
            label 'LOWTEST5'
          }
          steps {
            script {
              doWork('TQA_LOWTEST5')
            } 
          }
        }

        stage('MIDDLETEST1') {
          agent {
            label 'MIDDLETEST1'
          }
          steps {
            script {
              doWork('TQA_MIDDLETEST1')
            } 
          }
        }

        stage('MIDDLETEST2') {
          agent {
            label 'MIDDLETEST2'
          }
          steps {
            script {
              doWork('TQA_MIDDLETEST2')
            } 
          }
        }

        stage('MIDDLETEST3') {
          agent {
            label 'MIDDLETEST3'
          }
          steps {
            script {
              doWork('TQA_MIDDLETEST3')
            } 
          }
        }

        stage('MIDDLETEST4') {
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