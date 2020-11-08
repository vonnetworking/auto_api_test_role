pipeline {
  agent any
  parameters {
    text(name: 'DATA', defaultValue: null, description: 'json data payload')
  }
  stages {
    stage('Read Data') {
      steps {
        if {!params.DATA} {
          def jsonString = '{}'
        } else {
          def jsonString = params.data
          def jsonObj = readJSON text: jsonString
        }
        echo jsonObj
      }
    }
  }
}
