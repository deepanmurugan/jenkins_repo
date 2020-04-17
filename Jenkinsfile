pipeline {

 agent { node { label 'cloud' } }
 
    stages {
     
     stage('SCM Fetch') {
      steps {
        dir('/tmp/pipeline-project-1')
        {
          // Get some code from a GitHub repository
         checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: false, timeout: 10]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github-password', url: 'https://github.com/deepanmurugan/jenkins_repo.git']]]
        }
       }
      }
   
     stage('Build') {
      steps {
       // Run the build
       sh label: '', script: '''cd /tmp/pipeline-project-1
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
      steps {
        emailext attachLog: true, body: 'Check console output at $BUILD_URL to view the results.', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'opsworksss@noreply'
      }
      steps {
       s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'projcts-artifacts/${JOB_NAME}-${BUILD_NUMBER}', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: true, noUploadOnFailure: true, selectedRegion: 'us-east-2', showDirectlyInBrowser: false, sourceFile: '', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false, userMetadata: [[key: '', value: '']]]], pluginFailureResultConstraint: 'FAILURE', profileName: 'aws-s3-profile', userMetadata: []
     }
    }
  }
}
