
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

  def name = list[i] as String;

    def drive = "C";
    def stream = "Main"

    tests["${name}"] = {
      timeout(unit: 'MINUTES', time: 30 ) {
        node("${name}") {
          stage("${name}") {
            script {
              stage("준비") {
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
              stage('실행') {
                options += ' -ExecCmds="Automation RunTests BravoHotel.Utils.PSOCollectingTest"'


                  bat """
                  taskkill /f /im BravoHotel*
                  taskkill /f /im AutoHotKey*

                  net use \\\\oscarmike.io\\BravoHotel_Distribution ",q4W!q" /user:wonderpeople

                  set RUN_OPTIONS=${options}

                  pushd \\Auto && start AutoHotkey.exe check_crash.ahk ${name}
                  pushd ${drive}:\\Games\\RunGame_${stream} && RunGame_${stream}_Tqa.bat 
                  exit /b 0
                  """
              }
              stage('대기.') {
                def msg = powershell(returnStatus: true, script: 'Wait-Process -Name "BravoHotel*"')
                println msg
              }
              stage('정리.') {
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
    cron('TZ=Asia/Seoul\n0,30 14-21 * * *')
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