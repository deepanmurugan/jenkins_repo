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
    
    stage('Sonarqube scan code') {
      environment {
        scannerHome = tool 'sonar_jenkins'
       }
       steps {
       withSonarQubeEnv('sonar_jenkins') {
            sh "${scannerHome}/bin/sonar-scanner"
        timeout(time: 1, unit: 'HOURS')
         waitForQualityGate abortPipeline: true
        }
       }
     }
   
     stage('Build') {
      steps {
       // Run the build
       sh label: '', script: '''cd /tmp/pipeline-project-1
       jar -cvf samplenew.war *
       hostname -I'''
       echo 'Building the code'
       }
     }

     stage('Achive Artifacts') {
      steps {
       dir('/tmp/pipeline-project-1') {
          archiveArtifacts '*.war'
          echo 'Archive is completed'
        }
      }
     }
    
     stage('Post Build Actions') {
      steps {
        emailext attachLog: true, body: 'Check console output at $BUILD_URL and Approve/Reject.', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'deepan.gctpro@gmail.com'
      }
     }
      
     stage('Upload to S3') {
      steps {
       dir('/tmp/pipeline-project-1') {
       s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'projcts-artifacts/${JOB_NAME}-${BUILD_NUMBER}', excludedFile: '', flatten: true, gzipFiles: false, keepForever: false, managedArtifacts: true, noUploadOnFailure: true, selectedRegion: 'us-east-2', showDirectlyInBrowser: false, sourceFile: '*.war', storageClass: 'STANDARD', uploadFromSlave: true, useServerSideEncryption: false, userMetadata: [[key: '', value: '']]]], pluginFailureResultConstraint: 'FAILURE', profileName: 'aws-s3-profile', userMetadata: []
       }
      }
     }
      
     stage('Confirmation Stage') {
      agent none
      steps {
       input id: 'Pipeline_project_build', message: 'Do you want to proceed to production?', submitter: 'approver', parameters: [string(defaultValue: '', description: '', name: 'Any Comments to Approve/Reject', trim: true)]
      }
    }
     
     stage('Depoy to production') {
      steps {
       echo 'Deploying to Production'
      }
    }
      
  }
}
