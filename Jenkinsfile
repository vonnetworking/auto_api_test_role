pipeline {
  agent any
  parameters {
    text(name: 'DATA', defaultValue: '{}', description: 'json data payload')
  }
  stages {
    stage('Prep Env') {
      environment {
        workspaceTar = "${env.WORKSPACE_TMP}/workspace.tar.gz"
      }
      steps { //tar up the role and publish it into roles directory
        sh "tar -czf ${env.workspaceTar} ."
        dir ('roles/this_role') {
          sh "pwd"
          sh "tar -zxf ${env.workspaceTar}"
        }
      }
    }

    stage('Run Role') {
      environment {
        jsonObj = readJSON text: params.DATA;
        jsonList = jsonObj.values().toList()
      }
      steps { 
        
        ansiblePlaybook([
          playbook: jsonObj.playbook,
          extraVars: jsonObj.extraVars
        ])
       // ansiblePlaybook env.jsonObj
      }
    }
  }
}
