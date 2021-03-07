pipeline {
  agent any
  stages {
    stage('Run Ansible') {
      steps {
        library 'automation'
        script {
          runAnsible()
        }

        input(message: 'input_file', id: 'input_file')
      }
    }

  }
}