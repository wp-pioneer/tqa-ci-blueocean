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
          echo 'command dir ? and run bat '
          bat 'RunGame_Main_Test.bat'
        }

        bat 'cd \\Games\\RunGame_Main_Test'
        bat 'RunGame_Main_Test.bat'
      }
    }

  }
}