

//온라인 상태의 노드의 이름을 얻어온다.
//@NonCPS
def onlineNodeNames() { 
    String[] free_nodes = []
    for (Node node in jenkins.model.Jenkins.instance.nodes) {
        
        // PSO 머신은 테스트 잡에서 제외
        if( node.name.contains("PSO") ) {
          continue
        }

        //! 온라인 상태의 머신만 ( 모든 머신으로 할경우 대기 하는 경우 발생. )
        if (node != null && node.toComputer() != null && node.toComputer().online) {
          def computer = node.toComputer()

          if( computer.countBusy() <= 0 ) {
            free_nodes += node.name
            println "${computer.countBusy()}, ${computer.countIdle()}, ${computer.countExecutors()}";
          }
        }
    }
      //free_nodes = free_nodes.findAll { it != 'INTEL' }
    return free_nodes
}

//! 젠킨스 잡을 동적으로 생성한다. ( 동적으로 스크립트 제어 )
def doDynamicParallelSteps(){

  def list = onlineNodeNames();
  def totalCount = list.size();
  def inGameCount = 0;
  def isStarted = 0;

  tests = [:]
  for(int i=0; i < list.size(); i++) {

    def name = list[i] as String;
    //! 기본 드라이브 ( 모든 Test장비는 C/D 드라이브 포함 가정.)
    def drive = "D";

    //! 고사양와 INTEL PC는 C드라이브 ( SSD ) 로 실행 되도록 설정.
    if( name.contains("HIGH") || name.contains("INTEL")) {
      drive = "C";
    }

    tests["${name}"] = {
      //! 테스트 실행을 최대 50제한으로 한다. ( 50분 초과시 중단됨. )
      timeout(unit: 'MINUTES', time: 50 ) {
        node("${name}") {
          stage("${name}") {
            script {
              stage('prepare env') {
                def msg = powershell(returnStatus: true, script: '''$Process = Get-CimInstance -ClassName Win32_PRocess -Filter "CommandLine LIKE '%notepad%'"; Stop-Process -Id $Process.ProcessId -PassThru -Force''')
                println msg
              }
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
                //! 포트 번호를 빌드넘버마다 다르게 순환시킴.
                def port = 8801 + ( currentBuild.number % 3 ) 

                def isRunChangeSettings_GraphicOption = false

                //! 해당 머신만 그래픽 설정 테스트 옵션을 추가함.
                if(  ["HIGHTEST1","MIDDLETEST1","LOWTEST1"].contains(name) ) { 
                  isRunChangeSettings_GraphicOption = true 
                }

                //! 기본 실행 옵션.
                def options = '-ApiPhase="qa" -nosteam -ExecCmds="net.IpNetDriverReceiveThreadQueueMaxPackets 102400" -LogCmds="LogReplicationPingComponent All" -WithPingComponent -dx12 ServicePlatform="internal"'
                options += " -MonitoringEndThenRequestExit -performancemonitoring -nobenchmark -AutoJoinCmd=172.16.2.202:${port}?UserName=${name} 1 > nul" 


                if( isRunChangeSettings_GraphicOption ) {
                  options += ' GraphicOptionTestLoop=100 GraphicOptionTestInterval=5.0 -ExecCmds="Automation RunTests AutoUnitTests.ChangeSettings.GraphicOption"'
                }

                echo "running.... ${port}"

                  //! 배치 실행
                  bat """
                  taskkill /f /im BravoHotel*
                  taskkill /f /im AutoHotKey*

                  net use \\\\oscarmike.io\\BravoHotel_Distribution ",q4W!q" /user:wonderpeople
                  set USERNAME=${name}
                  set PORT=${port}
                  
                  set RUN_OPTIONS=${options} 

                  pushd \\Auto && start AutoHotkey.exe check_crash.ahk ${name}
                  pushd ${drive}:\\Games\\RunGame_QA && RunGame_QA_Tqa.bat 
                  exit /b 0
                  """
              }
              //! 실행후 업데이트가 있는 경우 대기한다 오토핫키 매크로로 이미지 체크
              stage('update') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_update2.ahk ${name}"
              }

              //! 실행후 게임 레디 이미지 체크후 추가 UE Commmand 입력한다. 입력은 stage_ready_game.ahk에 정의
              stage("inGame - ${inGameCount}") {
                echo "start ${params.AUTO_START}"
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_ready_game.ahk ${name} ${params.AUTO_START}"
                inGameCount += 1;
              }


              //! 노드들의 상태가 적당히 레디 카운드가 오르면 게임 시작 명령어를 날린다 ( stage_start_game4.ahk )에 정의 
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

              //! 게임 프로세스가 종료될때까지 대기한다.
              stage('waiting client...') {
                def msg = powershell(returnStatus: true, script: 'Wait-Process -Name "BravoHotel*"')
                println msg
              }
              
              //! 오토핫키 와 프로세스 정리 명령 
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
  //실행 주기 정의
  triggers {
    cron('TZ=Asia/Seoul\n0 5-8 * * *')
  }
  //! 실행시 성공/실패 슬랙 메시지 정의
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