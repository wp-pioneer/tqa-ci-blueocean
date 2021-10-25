pipeline {
  agent {
    node {
      label 'worker-leo'
    }

  }
  stages {
    stage('run games') {
      steps {
        dir(path: 'E:\\Games\\RunGame_Main_Test')
        bat(script: 'RunGame_Main_Test.bat', returnStatus: true, returnStdout: true)
      }
    }

  }
}