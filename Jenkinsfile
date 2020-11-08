pipeline {
  agent any
  parameters {
    text(name: 'DATA', defaultValue: '{}', description: 'json data payload')
  }
  stages {
    stage('Read Data') {
      steps {
        script {
          jsonString = params.data;
          jsonObj = readJSON text: jsonString;
        }
 
        echo jsonObj
      }
    }
  }
}
