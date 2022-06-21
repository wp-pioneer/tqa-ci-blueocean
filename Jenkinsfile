

//@NonCPS
def onlineNodeNames() {
    String[] free_nodes = []
     for (Node node in jenkins.model.Jenkins.instance.nodes) {
         // Make sure slave is online
         if (node != null && node.toComputer() != null && node.toComputer().online) {
           def computer = node.toComputer() 
           
           if( computer.countBusy() <= 0 ) {
             free_nodes += node.name
             println "${computer.countBusy()}, ${computer.countIdle()}, ${computer.countExecutors()}";
           }
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
    def drive = "D";
    if( name.contains("HIGH") || name.contains("INTEL")  ) {
      drive = "C";
    }
    tests["${name}"] = {
      timeout(unit: 'MINUTES', time: 50 ) {
        node("${name}") {
          stage("${name}") {
            script {
              stage("start") {
                echo "starting.. ${params.AUTO_START}"
                def computer = jenkins.model.Jenkins.instance.getComputer(env.NODE_NAME)

                echo "info - ${computer.countBusy()}"
                bat """
                taskkill /f /im BravoHotel*
                taskkill /f /im AutoHotKey*
                net use
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Auto c:\\Auto /MIR /s /TEE
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Games ${drive}:\\Games /s /TEE 
                exit /b 0
                """
              }
              stage('run') {
                def port = 8802 + ( currentBuild.number % 2)

                def isRunChangeSettings_GraphicOption = false

                if( ["HIGHTEST1","MIDDLETEST1","LOWTEST1"].contains(name) ) {
                  isRunChangeSettings_GraphicOption = true 
                }

                def options = '-SkipPatch -ApiPhase="dev2_for_dev_stream" -MatchMakingTag="GM_BattleRoyale_DEV" -GameMode="GM_BattleRoyale_DEV" -IgnoreCatalogue -dx12 ServicePlatform="internal" -SelectExec="BravoHotelGame\\Binaries\\Win64\\BravoHotelClient-Win64-Test.exe" -MonitoringEndThenRequestExit -performancemonitoring -nobenchmark'
                options += " -AutoJoinCmd=172.16.2.201:${port}?UserName=${name}" 


                if( isRunChangeSettings_GraphicOption ) {
                  options += ' GraphicOptionTestLoop=100 GraphicOptionTestInterval=5.0 -ExecCmds="Automation RunTests AutoUnitTests.ChangeSettings.GraphicOption"'
                }

                if( name.contains("INTEL") || name.contains("LOWTEST9") )  { 
                  options += ' -ExecCmds="Automation RunTests BravoHotel.Scenario.QAPerformance_Default"'
                }


                echo "running.... ${port}"

                  bat """
                  taskkill /f /im BravoHotel*
                  taskkill /f /im AutoHotKey*

                  net use \\\\oscarmike.io\\BravoHotel_Distribution ",q4W!q" /user:wonderpeople
                  set USERNAME=${name}
                  set PORT=${port}

                  set RUN_OPTIONS=${options}

                  pushd \\Auto && start AutoHotkey.exe check_crash.ahk ${name}
                  pushd ${drive}:\\Games\\RunGame_Dev && RunGame_Dev_Tqa.bat 
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
                if( isStarted == 0 && percent > 80 ) {
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
    cron('TZ=Asia/Seoul\n30 17-23 * * *')
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