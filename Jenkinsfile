
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
  def totalCount = list.size();
  def inGameCount = 0;
  def isStarted = 0;

  tests = [:]
  for(int i=0; i < list.size(); i++) {
    def name = list[i] as String;
    tests["${name}"] = {
      timeout(unit: 'MINUTES', time: 50 ) {
        node("${name}") {
          stage("${name}") {
            script {
              stage("start") {
                echo "starting.. ${params.AUTO_START}"
                bat """
                taskkill /f /im BravoHotel*
                taskkill /f /im AutoHotKey*
                cmdkey /list
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Auto c:\\Auto /MIR /s /TEE
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Games c:\\Games /s /TEE 
                exit /b 0
                """
              }
              stage('run') {
                def port = 8000 + ( currentBuild.number % 5)
               
                echo "running.... ${port}"

                  bat """
                  taskkill /f /im BravoHotel*
                  taskkill /f /im AutoHotKey*
                  
                  pushd \\Auto && start AutoHotkey.exe check_crash.ahk ${name}
                  pushd \\Games\\RunGame_Dev
                  net use \\oscarmike.io\BravoHotel_Distribution ",q4W!q" /user:wonderpeople
                  set ScriptsFolder=\\oscarmike.io\BravoHotel_Distribution\Scripts_QA\PackagedClient
                  set DestFolder=%cd%\Scripts

                  robocopy %ScriptsFolder% %DestFolder% /MIR

                  %cd%\Scripts\RunPackagedGame_QA_Tqa.bat ${name} ${port}
                  exit /b 0
                  """
              }
              stage('update') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_update2.ahk ${name}"
              }
              stage("inGame - ${inGameCount}") {
                echo "start ${params.AUTO_START}"
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_ready_game.ahk ${name} ${params.AUTO_START}"
                inGameCount += 1;
              }
              
              stage("start - ${inGameCount}") {
                def percent = ((inGameCount / totalCount ) * 100)
                if( isStarted == 0 && percent > 85 ) {
                  isStarted = 1
                  echo 'try start..'
                  sleep(180)
                  echo 'starting...'
                  bat "pushd \\Auto && start /w AutoHotkey.exe stage_start_game4.ahk ${name} ${params.AUTO_START}"
                } 
              }
              
              /*stage('returnLobby') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_return_lobby.ahk ${name}"
              }*/
              stage('waiting client...') {
                def msg = powershell(returnStatus: true, script: 'Wait-Process -Name "BravoHotel*"')
                println msg
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