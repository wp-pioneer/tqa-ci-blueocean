
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
              stage("@${name} start") {
                echo "starting.. ${params.AUTO_START}"
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
                pushd \\Auto && AutoHotkey.exe check_crash.ahk ${name}
                pushd \\Games\\RunGame_QA && RunGame_QA.bat
                exit /b 0
                """
              }
              stage('update') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_update.ahk ${name}"
              }
              stage('login') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_login.ahk ${name}"
              }
              stage('makeAccount') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_makeAccount.ahk ${name}"
              }
              stage('lobby') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_lobby.ahk ${name}"
              }
              stage('mode_select') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_mode_select.ahk ${name}"
              }
              stage('startGame') {
                echo 'start ${params.AUTO_START}'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_start_game.ahk ${name} ${params.AUTO_START}"
              }
              stage('returnLobby') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_return_lobby.ahk ${name}"
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
  triggers {
    cron('TZ=Asia/Seoul\n0 3-8 * * *')
  }
  stages {
    stage('input.') {
      steps {
        script {
          properties([
              parameters([
                  /*choice(
                      choices: ['ONE', 'TWO'], 
                      name: 'PARAMETER_01'
                  ),*/
                  booleanParam(
                      defaultValue: true, 
                      description: 'true 일경우 자동으로 시작, false 일경우 매칭만', 
                      name: 'AUTO_START'
                  ),
                  /*text(
                      defaultValue: '''
                      this is a multi-line 
                      string parameter example
                      ''', 
                        name: 'MULTI-LINE-STRING'
                  ),
                  string(
                      defaultValue: 'scriptcrunch', 
                      name: 'STRING-PARAMETER', 
                      trim: true
                  )*/
              ])
          ])
        }
      }
    }
    stage('병렬처리') {
      steps {
        script {
          doDynamicParallelSteps()
        }
      }
    }
  }
}