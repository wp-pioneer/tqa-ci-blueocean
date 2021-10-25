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
          steps {
            bat 'pushd \\Games\\RunGame_Main_Test && RunGame_Main_Test.bat'
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