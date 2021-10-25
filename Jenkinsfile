pipeline {
  agent none
  stages {
    stage('parallels stage') {

      parallel {
        stage('run1') {
          agent {
            label 'worker-leo'
          }
          steps {
            echo 'step: run game...'
            bat 'pushd \\Games\\RunGame_Main_Test && RunGame_Main_Test.bat'
            echo 'step: login'
            bat 'pushd \\Auto && AutoHotkey.exe login.ahk'
          }
        }

        stage('run2') {
          agent {
            label 'stella'
          }
          steps {
            echo 'step: run game...'
          }
        }

      }
    }

  }
}