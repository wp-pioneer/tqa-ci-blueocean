pipeline {
  agent {
    node {
      label 'worker-leo'
    }

  }
  stages {
    stage('run games') {
      steps {
        dir(path: '/Games/RunGame_Main_Test')
        bat(script: 'dir', returnStatus: true, returnStdout: true)
      }
    }

  }
}