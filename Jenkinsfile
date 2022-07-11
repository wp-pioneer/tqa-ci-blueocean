
//@NonCPS
def onlineNodeNames() {
    String[] free_nodes = []
     for (Node node in jenkins.model.Jenkins.instance.nodes) {
        if( node.name.contains("PSO") ) {
          continue
        }
         // Make sure slave is online
         if (node != null && node.toComputer() != null && node.toComputer().online) {
             free_nodes += node.name 
         }
     }
    return free_nodes
}

def doDynamicParallelSteps(){

  def list = onlineNodeNames();

  tests = [:]
  for(int i=0; i < list.size(); i++) {
    def name = list[i] as String;
     def drive = "D";
    if( name.contains("HIGH") || name.contains("INTEL")) {
      drive = "C";
    }
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
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Auto c:\\Auto /MIR /s /TEE
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Games ${drive}:\\Games /s /TEE 
                exit /b 0
                """
              }

              stage('run') {
                echo 'running....'
                  bat """
                  taskkill /f /im BravoHotel*
                  taskkill /f /im AutoHotKey*
                  net use \\\\oscarmike.io\\BravoHotel_Distribution ",q4W!q" /user:wonderpeople
                  set RUN_OPTIONS=-ApiPhase="dev2_for_dev_stream" -MatchMakingTag="GM_BattleRoyale_DEV" -GameMode="GM_BattleRoyale_DEV" -IgnoreCatalogue -dx12 ServicePlatform="internal" -SelectExec="BravoHotelGame\\Binaries\\Win64\\BravoHotelClient-Win64-Test.exe" -PatchEndThenRequestExit -MonitoringEndThenRequestExit -nobenchmark
                  pushd ${drive}:\\Games\\RunGame_Dev && RunGame_Dev_Tqa.bat ${name}
                  exit /b 0
                  """
              }
              stage('update') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_update2.ahk ${name}"
              }
              /*stage('login') {
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
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_start_game3.ahk ${name} ${params.AUTO_START}"
              }
              stage('returnLobby') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_return_lobby.ahk ${name}"
              }*/
              stage('cleanup') {
                echo 'cleanup11'
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
    overrideIndexTriggers(false)
    disableConcurrentBuilds()
    preserveStashes(buildCount: 10)
  }
  triggers {
    cron('TZ=Asia/Seoul\n0 10 * * *')
  }
  post {
    success {
        slackSend (channel: '#tqa-ci', color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
    failure {
        slackSend (channel: '#tqa-ci', color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
  }
  stages {
    stage('start') {
      steps { slackSend (channel: '#tqa-ci', color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})") }
    }
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