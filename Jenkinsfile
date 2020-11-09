pipeline {
  agent any
  parameters {
    text(name: 'DATA', defaultValue: '{}', description: 'json data payload')
  }
  stages {
    stage('Prep Env') {
      steps {
        dir ('roles') {
          sh "rsync -rv --exclude=roles --exclude=.git .. ."
        }
      }
    }

    stage('Read Data') {
      environment {
        jsonObj = readJSON text: params.DATA;
        extraVars = "$jsonObj.extraVars"
      }
      steps { 
        echo env.jsonObj
      }
    }
  }
}
