pipeline {
  agent {
    node {
      label 'worker-leo'
    }
  }
  stages {
    stage('run games') {
      steps {
        bat 'cd \\ && dir '
        bat 'RunGame_Main_Test.bat'
      }
    }

  }
}