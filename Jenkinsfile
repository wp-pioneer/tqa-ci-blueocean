pipeline {
  agent {
    node {
      label 'worker-leo'
    }

  }
  stages {
    stage('run games') {
      steps {
        bat 'cd \\Games\\RunGame_Main_Test'
        bat 'RunGame_Main_Test.bat'
        powershell 'cd'
      }
    }

  }
}