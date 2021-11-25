def doDynamicParallelSteps(){
  def list = ["HIGHTEST1", "HIGHTEST2", "HIGHTEST3", "HIGHTEST4", 
              "LOWTEST1", "LOWTEST2", "LOWTEST3", "LOWTEST4", "LOWTEST5",
              "MIDDLETEST1", "MIDDLETEST2", "MIDDLETEST3", "MIDDLETEST4"]              
  tests = [:]
  for(int i=0; i < list.size(); i++) {
    def name = list[i] as String;
    tests["${name}"] = {
      timeout(unit: 'MINUTES', time: 50 ) {
        node("${name}") {
          stage("${name}") {
            script {
              stage('cleanup') {
                echo 'cleanup.... kill all'
                bat """
                taskkill /f /im robocopy*
                taskkill /f /im BravoHotel*
                taskkill /f /im AutoHotKey*
                exit /b 0
                """
              }
            }
          }
        }
      }
    }
  }

  parallel tests
}


pipeline {
  agent none
  options {
    disableConcurrentBuilds()
    preserveStashes(buildCount: 10)
  }
  stages {
    stage('parallels steps') {
      steps {
        script {
          doDynamicParallelSteps()
        }
      }
    }
  }
}