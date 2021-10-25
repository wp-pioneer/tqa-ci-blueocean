pipeline {
  agent {
    node {
      label 'worker-leo'
    }

  }
  stages {
    stage('error') {
      steps {
        bat(script: 'notepad.exe', encoding: 'utf-8', returnStatus: true, returnStdout: true)
      }
    }

  }
}