pipeline {
  agent {
    docker {
      image 'ubuntu'
    }
    
  }
  stages {
    stage('step1') {
      steps {
        powershell(script: 'git status', returnStatus: true, returnStdout: true)
      }
    }
  }
}