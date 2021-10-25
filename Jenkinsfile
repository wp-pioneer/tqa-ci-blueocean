pipeline {
  agent {
    node {
      label 'worker-leo'
    }

  }
  stages {
    stage('run games') {
      steps {
        bat(script: 'RunGame_Main_Test.bat', returnStatus: true, returnStdout: true)
      }
    }

  }
}