pipeline {
  agent {
    node {
      label 'worker-leo'
    }
  }
  stages {
    stage('parallels stage') {

      parallel { 

        stages('run-worker-leo') {
          agent { label 'worker-leo'}
          stage('run1') {
            steps {
              bat 'pushd \\Games\\RunGame_Main_Test && RunGame_Main_Test.bat'
            }
          }
          stage('waiting login ui') {
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