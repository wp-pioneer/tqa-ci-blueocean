pipeline {
  agent {
    node {
      label 'worker-leo'
    }

  }
  stages {
    stage('run games') {
      steps {
        dir(path: '\\Games\\RunGame_Main_Test') {
          echo 'change current dir'
        }

        bat 'cmd /c dir'
        bat 'RunGame_Main_Test.bat'
        powershell 'cd'
      }
    }

  }
}