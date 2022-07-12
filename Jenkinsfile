
//@NonCPS
def onlineNodeNames() {
    String[] free_nodes = []
     for (Node node in jenkins.model.Jenkins.instance.nodes) {
         if (node != null && node.toComputer() != null && node.toComputer().online) {
           if( node.name.contains("PSO") ) {
             free_nodes += node.name 
           }
         }
     }
    return free_nodes
}

def doDynamicParallelSteps(){

  def list = onlineNodeNames();

  def stream = "Main"
  def WS = "jenkins_dll_PSO_Main"
  def drive = "C"

  tests = [:]
  for(int i=0; i < list.size(); i++) {
    def name = list[i] as String;
    tests["${name}"] = {
      timeout(unit: 'MINUTES', time: 30 ) {
        node("${name}") {
          stage("${name}") {
            script {
              stage('prepare') {
                bat """
                taskkill /f /im BravoHotel*
                taskkill /f /im AutoHotKey*
                net use \\\\oscarmike.io\\BravoHotel_Distribution ",q4W!q" /user:wonderpeople

                exit /b 0
                """
              }
              stage('PSO 수집') {
                bat """
                ${drive}:
                pushd \\
                robocopy \\\\oscarmike.io\\Shared\\PSO\\${stream} \\PSOCaching /MIR /s /TEE

                exit /b 0
                """
              }
              stage('에디터 업데이트') {
                bat """
                ${drive}
                pushd \\Workspaces\\${WS}\\ZBravoHotel
                SyncLatestEditor.bat
                exit /b 0
                """
              }
              stage('PSO 머지') {
                bat """
                echo "copy prev pso cache"
                copy ${drive}:\\Workspaces\\${WS}\\ZBravoHotel\\Build\\Windows\\PipelineCaches\\BravoHotelGame_PCD3D_SM5.stable.upipelinecache \\PSOCaching /y

                echo "delete prev merged cache"
                del ${drive}:\\PSOCaching\\BravoHotelGame_PCD3D_SM5.upipelinecache

                echo "merge pso cache"
                pushd \\Workspaces\\${WS}\\ZBravoHotel
                ..\\Engine\\Binaries\\Win64\\UE4Editor-Cmd.exe BravoHotelGame -run=ShaderPipelineCacheTools Merge ${drive}:\\PSOCaching\\*.upipelineCache ${drive}:\\PSOCaching\\BravoHotelGame_PCD3D_SM5.upipelinecache

                echo "robocopy"
                robocopy ${drive}:\\PSOCaching ${drive}:\\Games\\RunGame_${stream}\\BravoHotelGameApp\\MinApp\\WindowsClient\\BravoHotelGame\\Saved BravoHotelGame_PCD3D_SM5.upipelinecache /MOV

                exit /b 0
                """
              }

              stage('클라이언트 실행'){
                bat """
                set RUN_OPTIONS=-nostablepipelinecache -ExecCmds="Automation RunTests BravoHotel.RenderCore.SaveShaderPipelineCache"
                pushd ${drive}:\\Games\\RunGame_${stream} && RunGame_${stream}_Tqa.bat
                exit /b 0

                """
              }

              stage('대기') {
                sleep 10
                bat """
                :LOOP
                tasklist | find /i "BravoHotel" >nul 2>&1
                IF ERRORLEVEL 1 (
                  exit /b 0
                ) ELSE (
                  ECHO still running
                  GOTO LOOP
                )
                exit /b 0
                """
              }

              stage('P4업데이트') {
                bat """
                p4 set P4USER=jenkins_dll
                p4 set P4PORT=p4.oscarmike.io:1666
                p4 set P4PASSWD=%P4_JENKINS_PW%
                p4 set P4CHARSET=utf8
                p4 set P4CLIENT=${WS}
                p4 info
                p4 sync -f ${drive}:\\Workspaces\\${WS}\\ZBravoHotel\\Build\\Windows\\PipelineCaches\\BravoHotelGame_PCD3D_SM5.stable.upipelinecache 
                p4 edit ${drive}:\\Workspaces\\${WS}\\ZBravoHotel\\Build\\Windows\\PipelineCaches\\BravoHotelGame_PCD3D_SM5.stable.upipelinecache 
                copy ${drive}:\\Games\\RunGame_${stream}\\BravoHotelGameApp\\MinApp\\WindowsClient\\BravoHotelGame\\Saved\\BravoHotelGame_PCD3D_SM5.upipelinecache ${drive}:\\Workspaces\\${WS}\\ZBravoHotel\\Build\\Windows\\PipelineCaches\\BravoHotelGame_PCD3D_SM5.stable.upipelinecache /y
                p4 submit -d "PSO CACHE UPLOAD" 
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
    cron('TZ=Asia/Seoul\n30 10 * * *')
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
    stage('병렬처리') {
      steps {
        script {
          doDynamicParallelSteps()
        }
      }
    }
  }
}