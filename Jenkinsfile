
//@NonCPS
def onlineNodeNames() {
    String[] free_nodes = []
     for (Node node in jenkins.model.Jenkins.instance.nodes) {
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
                net use %RELEASE_FOLDER% ",q4W!q" /user:wonderpeople
                cmdkey /list
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Auto c:\\Auto /MIR /s /TEE
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Games c:\\Games /s /TEE 
                exit /b 0
                """
              }
              stage('run') {
                echo 'running....'
                  bat """
                  taskkill /f /im BravoHotel*
                  taskkill /f /im AutoHotKey*
                  pushd \\Games\\RunGame_Dev && RunGame_Dev_Tqa_nopatch.bat ${name}
                  pushd \\Auto && start AutoHotkey.exe check_crash.ahk ${name}
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
              }*/
              stage('startGame') {
                echo 'start ${params.AUTO_START}'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_start_game3.ahk ${name} ${params.AUTO_START}"
              }
              /*stage('returnLobby') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_return_lobby.ahk ${name}"
              }*/
              stage('waiting client...') {
                 bat "\\Games\\WaitBravoHotelProcess.bat"
              }
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
    cron('TZ=Asia/Seoul\n0 11-23 * * *')
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