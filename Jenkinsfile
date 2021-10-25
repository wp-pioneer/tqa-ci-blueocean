pipeline {
  agent {
    node {
      label 'worker-leo'
    }
  }
  stages {
    stage('run games') {
      steps {
        bat 'pushd \\Games\RunGame_Main_Test && RunGame_Main_Test.bat'
      }
    }

  }
}