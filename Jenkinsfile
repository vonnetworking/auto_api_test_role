pipeline {
  agent any
  parameters {
    text(name: 'DATA', defaultValue: '{}', description: 'json data payload')
  }
  stages {
    stage('Read Data') {
      steps {
        script {
          def jsonString = params.data
          def jsonObj = readJSON text: jsonString
        }
        echo jsonObj
      }
    }
  }
}
