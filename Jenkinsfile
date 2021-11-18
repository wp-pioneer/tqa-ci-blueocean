def doWork( final String name) {
  echo 'step: run game...'
  bat 'pushd \\Games\\RunGame_QA && RunGame_QA_Test.bat'
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