import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

pipeline {
  agent any
  
  environment {
    AZURE_SUBSCRIPTION_ID = 'b0871bad-f323-4924-9f73-7083cde9f58d'
    AZURE_TENANT_ID = '0fbcfe2b-5ff0-4827-ae9f-66c2a3b2394e'
  }
  
  stages {
    stage('init') {
      steps {
        checkout scm
      }
    }
    
    stage('build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage('deploy') {
      steps {
        script {
          def resourceGroup = 'Mynewrg1'
          def webAppName = 'mydbapps'
          
          // login Azure
          withCredentials([azureServicePrincipal('admin12')]) {
    sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
}
          
          // get publish settings
          def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
          def ftpProfile = getFtpPublishProfile(pubProfilesJson)
          
          // upload package
          sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
          
          // log out
          sh 'az logout'
        }
      }
    }
  }
}
