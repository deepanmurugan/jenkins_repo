pipeline {

 agent { node { label 'cloud' } }
 
  environment { 
    github_repo = 'https://github.com/deepanmurugan/jenkins_repo.git'
    github_branch = '*/master'
    build_path = '/tmp/${BUILD_NAME}'
  }
    stages {
     
     stage('SCM Fetch') {
      steps {
        dir('/tmp/pipeline-project-1')
        {
          // Get some code from a GitHub repository
         checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: ${github_branch}]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: false, timeout: 10]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github-password', url: ${github_repo}]]]
        }
       }
      }
   
     stage('Build') {
      steps {
       // Run the build
       sh label: '', script: '''cd ${build_path}
       ls -ltr
       hostname -I'''
       echo 'Building the code'
       }
     }

     stage('Achive Artifacts') {
      steps {
       dir('/tmp/pipeline-project-1') {
          archiveArtifacts '*'
          echo 'Archive is completed'
        }
      }
     }
    
     stage('Post Build Actions') {
      steps('Email All')
      {
        emailext attachLog: true, body: 'Check console output at $BUILD_URL to view the results.', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'deepan.gctpro@gmail.com'
      }
     }
  }
}
