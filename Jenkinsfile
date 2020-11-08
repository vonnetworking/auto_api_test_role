pipeline {
  agent any
  parameters {
    text(name: 'DATA', defaultValue: '{}', description: 'json data payload')
  }
  stages {
    stage('Read Data') {
      environment {
        jsonObj = readJSON text: params.data;
      }
      steps { 
        echo env.jsonObj
      }
    }
  }
}
