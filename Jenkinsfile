pipeline {

 agent { 
  node { 
   label 'cloud'
   customWorkspace '/tmp/multibranch'
  } 
 }
 
   stages {
     
     stage('SCM Fetch') {
      steps {
          // Get some code from a GitHub repository
       echo "${JOB_NAME}"
         checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: false, timeout: 10]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github-password', url: 'https://github.com/deepanmurugan/jenkins_repo.git']]]
         dir('/tmp/ansible-playbooks/') {
          checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: false, timeout: 10]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github-password', url: 'https://github.com/deepanmurugan/Ansible_Playbook.git']]]
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
        }
        timeout(time: 1, unit: 'HOURS') {
         waitForQualityGate abortPipeline: true
        }
      }
    }
   
     stage('Build') {
      steps {
       // Run the build
       sh label: '', script: '''jar -cvf samplenew.war *
       hostname -I'''
       echo 'Building the code'
       }
     }

     stage('Achive Artifacts') {
      steps {
          archiveArtifacts '*.war'
          echo 'Archive is completed'
        }
     }
    
     stage('Post Build Actions - Email for Approval') {
      steps {
        emailext attachLog: true, body: 'Check console output at $BUILD_URL and Approve/Reject.', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'deepan.gctpro@gmail.com'
      }
     }
      
     stage('Approve the build to stage?') {
      agent none
      steps {
       input id: 'Pipeline_project_build', message: 'Do you want to proceed to production?', submitter: 'approver', parameters: [string(defaultValue: '', description: '', name: 'Any Comments to Approve/Reject', trim: true)]
      }
    }
    
     stage('Upload Artifacts to S3') {
      steps {
       s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'projcts-artifacts/${JOB_NAME}-${BUILD_NUMBER}', excludedFile: '', flatten: true, gzipFiles: false, keepForever: false, managedArtifacts: true, noUploadOnFailure: true, selectedRegion: 'us-east-2', showDirectlyInBrowser: false, sourceFile: '*.war', storageClass: 'STANDARD', uploadFromSlave: true, useServerSideEncryption: false, userMetadata: [[key: '', value: '']]]], pluginFailureResultConstraint: 'FAILURE', profileName: 'aws-s3-profile', userMetadata: []
       }
     }
     
     stage('Depoying to Staging') {
      steps {
       sh label: '', script: 'sudo apt-get install ansible -y'
       ansiblePlaybook become: true, extras: '"-e ansible_python_interpreter=/usr/bin/python3 jobname=${JOB_NAME} branchname=${BRANCH_NAME} buildno=${BUILD_NUMBER}"', credentialsId: 'deepan-ssh-access-key', disableHostKeyChecking: true, installation: 'ansible', inventory: '/tmp/ansible-playbooks/inv.ini', playbook: '/tmp/ansible-playbooks/install_war.yml'
       echo 'Deploying to Production'
      }
     }
    
  }
}
