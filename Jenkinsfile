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
  pushd \\Games\\RunGame_Main && RunGame_Main_Test.bat
  exit /b 0
  """
  echo 'step: login'
  bat "pushd \\Auto && AutoHotkey.exe example.ahk ${name}"
  //echo 'step: upload log'
  //bat 'pushd \\Auto && py pakinfo_upload.py'
}

def funcTest( final String name ) {
  echo "Element: ${name}"
}


def doDynamicParallelSteps(){
  def list = ["HIGHTEST1", "HIGHTEST2", "HIGHTEST3", "HIGHTEST4", 
              "LOWTEST1", "LOWTEST2", "LOWTEST3", "LOWTEST4", "LOWTEST5",
              "MIDDLETEST1", "MIDDLETEST2", "MIDDLETEST3", "MIDDLETEST4"]

  tests = [:]
  for(int i=0; i < list.size(); i++) {
    def name = list[i] as String;
    tests["${name}"] = {
      node("${name}") {
        timeout(unit: 'MINUTES', time: 50 ) {
          stage("${name}") {
            script {
              stage("@${name} start") {
                echo 'starting..'
                bat """
                taskkill /f /im BravoHotel*
                taskkill /f /im AutoHotKey*
                setlocal EnableDelayedExpansion
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Auto c:\\Auto /MIR /s /TEE
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Games c:\\Games /s /TEE 
                exit /b 0
                """
              }
              stage('run') {
                echo 'running....'
                bat """
                setlocal EnableDelayedExpansion
                taskkill /f /im BravoHotel*
                taskkill /f /im AutoHotKey*
                pushd \\Auto && start AutoHotkey.exe check_crash.ahk ${name}
                pushd \\Games\\RunGame_QA && RunGame_QA_Test.bat
                exit /b 0
                """
              }
              stage('update') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_update.ahk ${name}"
              }
              stage('dumpAllAreaRenderOverview') {
                echo 'lets go'
                bat "pushd \\Auto && AutoHotkey.exe dumpAllAreaRenderOverview.ahk ${name}"
              }
              stage('cleanup') {
                echo 'plz'
                bat """
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
  //triggers {
    //cron('TZ=Asia/Seoul\n0 3-8 * * *')
  //}
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