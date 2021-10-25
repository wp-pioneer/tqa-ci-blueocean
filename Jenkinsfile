pipeline {
  agent {
    node {
      label 'worker-leo'
    }

  }
  stages {
    stage('run games') {
      steps {
        bat(script: 'E:\\Games\\RunGame_Main_Test\\RunGame_Main_Test.bat', encoding: 'utf-8', returnStatus: true, returnStdout: true)
      }
    }

  }
}