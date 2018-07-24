def postActions(buildStatus) {
    def status

    switch (buildStatus) {
        case 'SUCCESS':
            sh 'vagrant destroy -f'
            status = 'is now green'
            break

        case 'UNSTABLE':
            status = 'is now unstable..'
            break

        case 'FAILURE':
            status = 'has turned RED :('
            break
    }
    cleanWs()
}

pipeline {

  agent { label 'master' }

  environment {
    RUN_TEST = true
    RUN_REMOTE = false
  }

  stages {
    stage('Get Ansible Config from Git') {
      steps {
        withCredentials([usernameColonPassword(credentialsId: 'dan_git_creds', variable: 'GITHUB_AUTH')]) {       
          sh "git clone https://${GITHUB_AUTH}@github.com/dani4571/cloud_snitch_vagrant.git --branch jenkins_test"
        }
      }
    }
    stage('Build') {
      steps{
        withCredentials([string(credentialsId: 'RAX_USERNAME', variable: 'RAX_USERNAME'),
                         string(credentialsId: 'RAX_API_KEY', variable: 'RAX_API_KEY'),
                         string(credentialsId: 'RAX_REG', variable: 'RAX_REG'),
                         string(credentialsId: 'RAX_USERNAME', variable: 'RS_USERNAME'),
                         string(credentialsId: 'RAX_API_KEY', variable: 'RS_API_KEY'),]) {
          sh 'export RS_REGION_NAME=DFW; rack files object download --container cloud-snitch-test-data --name james-newton-aio > james-newton-aio'
          sh 'vagrant up --provider=rackspace'
        }
      }
    }
  }

  post { 
    always { 
      withCredentials([string(credentialsId: 'RAX_USERNAME', variable: 'RAX_USERNAME'),
                         string(credentialsId: 'RAX_API_KEY', variable: 'RAX_API_KEY'),
                         string(credentialsId: 'RAX_REG', variable: 'RAX_REG')]) {

        postActions(currentBuild.currentResult)
      }
    }
  }
}
