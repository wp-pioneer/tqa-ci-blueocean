pipeline {
  agent {
    node {
      label 'worker-leo'
    }

  }
  stages {
    stage('run games') {
      steps {
        bat(script: 'E:\\Games\\RunGame_Main_Test\\RunGame_Main_Test.bat', returnStatus: true, returnStdout: true)
      }
    }

  }
}