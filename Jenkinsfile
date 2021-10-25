pipeline {
  agent {
    node {
      label 'worker-leo'
    }
  }
  stages {
    stage('parallels stage') {

      parallel { 
        stage('run1') {
          agent { label 'worker-leo'}
            echo('step: run game...')
            steps {
              bat 'pushd \\Games\\RunGame_Main_Test && RunGame_Main_Test.bat'
            }
            echo('step: login')
            steps {
              bat 'pushd \\Auto && AutoHotkey.exe login.ahk'
            }
          }
        }

        stage('run2') {
          agent { label 'stella'}
          steps {
            bat 'pushd \\Games\\RunGame_Main_Test && dir'
          }
        }

        stage('run3') {
          steps {
            bat 'pushd \\Games\\RunGame_Main_Test'
          }
        }

        stage('run4') {
          steps {
            bat 'pushd \\Games\\RunGame_Main_Test'
          }
        }


      }
    }

  }
}