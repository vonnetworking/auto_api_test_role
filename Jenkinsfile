pipeline {
  agent any
  stages {
    stage('Run Ansible') {
      steps {
        library 'automation'
        script {
          runAnsible();
        }

      }
    }

  }
}