def doWork() {
  echo 'step: run game...'
  bat 'pushd \\Games\\RunGame_QA && RunGame_QA_Test.bat'
  echo 'step: login'
  bat 'pushd \\Auto && AutoHotkey.exe example.ahk'
  echo 'step: upload log'
  bat 'pushd \\Auto && py pakinfo_upload.py'
}

def doWorkByLabel( final String label ) {
  stage("run ${label}") {
    agent {
      label : label
    }
    steps {
      echo 'step: run game...'
      bat 'pushd \\Games\\RunGame_QA && RunGame_QA_Test.bat'
      echo 'step: login'
      bat 'pushd \\Auto && AutoHotkey.exe example.ahk'
      echo 'step: upload log'
      bat 'pushd \\Auto && py pakinfo_upload.py'  
    }
  } 
}

pipeline {
  agent none
  stages {
    stage('parallels stage') {
      parallel {
        1: { doWorkByLabel('HIGHTEST1') },
        2: { doWorkByLabel('HIGHTEST2') },
        3: { doWorkByLabel('HIGHTEST3') },
        4: { doWorkByLabel('HIGHTEST4') },
      }
      
    }

  }
}