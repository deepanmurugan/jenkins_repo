pipeline {

 agent { label: 'cloud' }
 
    stages {
     
     stage('SCM Fetch') {
       dir('/tmp/pipeline-project-1')
       {
         // Get some code from a GitHub repository
         checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: false, timeout: 10]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github-password', url: 'https://github.com/deepanmurugan/jenkins_repo.git']]]
       }
      }
   
     stage('Build') {
       // Run the build
       sh label: '', script: '''cd /tmp/pipeline-project-1/
       ls -ltr
       hostname -I'''
       echo 'Building the code'
     }
   
     dir('/tmp/pipeline-project-1')
     {
        stage('Achive Artifacts') {
          archiveArtifacts '*'
          echo 'Archive is completed'
        }
     }
    
     stage('Post Build Actions')
     {
        stage('Email All')
        {
         emailext attachLog: true, body: 'Check console output at $BUILD_URL to view the results.', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'deepan.gctpro@gmail.com'
        }
     }
  }
}
