
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

  tests = [:]
  for(int i=0; i < list.size(); i++) {
    def name = list[i] as String;
    drive = "C";
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
              stage('copy pso') {
                bat """
                c:
                pushd \\
                robocopy \\\\oscarmike.io\\Shared\\PSO \\PSOCaching /MIR /s /TEE

                exit /b 0
                """
              }
              stage('sync editor') {
                bat """
                c:
                pushd \\Workspaces\\jenkins_dll_PSO_Main\\ZBravoHotel
                SyncLatestEditor.bat
                exit /b 0
                """
              }
              stage('merge cache') {
                bat """
                echo "copy prev pso cache"
                copy c:\\Workspaces\\jenkins_dll_PSO_Main\\ZBravoHotel\\Build\\Windows\\PipelineCaches\\BravoHotelGame_PCD3D_SM5.stable.upipelinecache \\PSOCaching /y

                echo "delete prev merged cache"
                del c:\\PSOCaching\\BravoHotelGame_PCD3D_SM5.upipelinecache

                echo "merge pso cache"
                pushd \\Workspaces\\jenkins_dll_PSO_Main\\ZBravoHotel
                ..\\Engine\\Binaries\\Win64\\UE4Editor-Cmd.exe BravoHotelGame -run=ShaderPipelineCacheTools Merge C:\\PSOCaching\\*.upipelineCache C:\\PSOCaching\\BravoHotelGame_PCD3D_SM5.upipelinecache

                echo "robocopy"
                robocopy C:\\PSOCaching C:\\Games\\RunGame_Main\\BravoHotelGameApp\\MinApp\\WindowsClient\\BravoHotelGame\\Saved BravoHotelGame_PCD3D_SM5.upipelinecache /MOV

                exit /b 0
                """
              }

              stage('run client with merge command'){
                bat """
                set RUN_OPTIONS=-nostablepipelinecache -ExecCmds="Automation RunTests BravoHotel.RenderCore.SaveShaderPipelineCache"
                pushd C:\\Games\\RunGame_Main && RunGame_Main_Tqa.bat
                exit /b 0

                """
              }

              stage('wait client') {
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

              stage('update cache') {
                bat """
                p4 info
                copy C:\\Games\\RunGame_Main\\BravoHotelGameApp\\MinApp\\WindowsClient\\BravoHotelGame\\Saved\\BravoHotelGame_PCD3D_SM5.upipelinecache C:\\Workspaces\\jenkins_dll_PSO_Main\\ZBravoHotel\\Build\\Windows\\PipelineCaches\\BravoHotelGame_PCD3D_SM5.stable.upipelinecache /y
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