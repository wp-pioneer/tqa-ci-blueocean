def doWork( final String name) {
  echo 'startingg...'
  echo 'syncing macro script...'
  bat """
  setlocal EnableDelayedExpansion
  robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Auto c:\\Auto /MIR /s /TEE
  exit /b 0
  """
  echo 'start main script'
  bat """
  setlocal EnableDelayedExpansion
  taskkill /f /im BravoHotel*
  taskkill /f /im AutoHotKey*
  pushd \\Games\\RunGame_QA && RunGame_QA_Test.bat
  exit /b 0
  """
  echo 'step: login'
  bat "pushd \\Auto && AutoHotkey.exe example.ahk ${name}"
  //echo 'step: upload log'
  //bat 'pushd \\Auto && py pakinfo_upload.py'
}

def funcTest( final String name ) {
  echo "Element: $name"
}

def doDynamicParallelSteps(){
  def list = ["HIGHTEST1", "HIGHTEST2", "HIGHTEST3", "HIGHTEST4", 
              "LOWTEST1", "LOWTEST2", "LOWTEST3", "LOWTEST4", "LOWTEST5",
              "MIDDLETEST1", "MIDDLETEST2", "MIDDLETEST3", "MIDDLETEST4"]              
  tests = [:]
  for(int i=0; i < list.size(); i++) {
    tests["${list[i]}"] = {
      node {
        label "${list[i]}"
        stage("${list[i]}") {
          script {
            funcTest("${list[i]}")
          }
        }
      }
    }
  }

  parallel tests
}


pipeline {
  agent none
  triggers {
    cron('0 */1 * * *')
  }
  stages {
    stage('병렬처리') {
      steps {
        script {
          doDynamicParallelSteps()
        }
      }
    }
  }
}