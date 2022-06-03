

//@NonCPS
def onlineNodeNames() {
    String[] free_nodes = []
     for (Node node in jenkins.model.Jenkins.instance.nodes) {
         // Make sure slave is online
        if( ['HIGHTEST4'].contains( node.name) ) {
          if (node != null && node.toComputer() != null && node.toComputer().online) {
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
                robocopy \\\\kate.oscarmike.io\\SharedDDC\\kate\\Games D:\\Games /s /TEE 
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
                pushd D:\\Games\\RunGame_Dev && RunGame_Dev_Test_assetmonitoring.bat
                exit /b 0
                """
              }
              /*stage('update') {
                echo 'plz'
                bat "pushd \\Auto && start /w AutoHotkey.exe stage_update.ahk ${name}"
              }
              stage('dumpAllAreaRenderOverview') {
                echo 'lets go'
                bat "pushd \\Auto && AutoHotkey.exe dumpAllAreaRenderOverview.ahk ${name} QA"
              }*/
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
    overrideIndexTriggers(false)
    disableConcurrentBuilds()
    preserveStashes(buildCount: 10)
  }
  triggers {
    cron('TZ=Asia/Seoul\n0 0 * * *')
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