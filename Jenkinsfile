pipeline {
  agent {
    node {
      label 'worker-leo'
    }
  }
  stages {
    stage('run games') {
      steps {
        bat 'cd \\'
        bat 'RunGame_Main_Test.bat'
      }
    }

  }
}