pipeline {
  agent any
  parameters {
    text(name: 'DATA', defaultValue: '{}', description: 'json data payload')
  }
  stages {
    stage('Read Data') {
      def jsonString = params.data
      def jsonObj = readJSON text: jsonString
      echo jsonObj
    }
  }
}
