 properties([[$class: 'JiraProjectProperty', siteName: 'https://lovescloud.atlassian.net/'], pipelineTriggers([githubPush()])])
 node {
   //try {
    // notifyBuild('STARTED')
 
     /* ... existing build steps ... */
     def mvnHome
   def scannerHome
   stage('Prepare') {
      cleanWs disableDeferredWipeout: true, notFailBuild: true
      git branch: 'develop', url: 'https://github.com/LovesCloud/RIL-Workshop.git'           
      mvnHome = tool 'M3'
      def commit = sh(returnStdout: true, script: 'git log -1 --pretty=%B | cat')
      //def matcher = commit =~ ([a-zA-Z][a-zA-Z0-9_]+-[1-9][0-9]*)([^.]|\.[^0-9]|\.$|$)
      print commit
      //print matcher
      scannerHome = tool 'sonar_scanner';
   }

   stage('Build') {
      sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
        
   }
   
   stage('Test-JUnit') {
      sh "'${mvnHome}/bin/mvn' test surefire-report:report"
   }
   
   stage('Sonar') {
      withSonarQubeEnv('SonarQube') {
        sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectName=RIL-Workshop -Dsonar.projectKey=RILW -Dsonar.sources=src -Dsonar.java.binaries=target/"
      }
   }


   stage('Docker-Build') {
      sh"""#!/bin/bash
         docker build . -t crud-mysql-vuejs:${BUILD_NUMBER}
      """
   }

   stage('Docker-Push') {
      /*withDockerRegistry(credentialsId: 'nexus', url: 'http://nexus.loves.cloud:8083') {
          sh"""#!/bin/bash
             docker tag crud-mysql-vuejs:${BUILD_NUMBER} nexus.loves.cloud:8083/crud-mysql-vuejs:${BUILD_NUMBER}
             docker push nexus.loves.cloud:8083/crud-mysql-vuejs:${BUILD_NUMBER}
             docker tag  nexus.loves.cloud:8083/crud-mysql-vuejs:${BUILD_NUMBER} crud-mysql-vuejs:${BUILD_NUMBER}
          """
      }*/

      withDockerRegistry(credentialsId: 'dockerhub') {
         sh"""#!/bin/bash
             docker tag crud-mysql-vuejs:${BUILD_NUMBER} lovescloud/crud-mysql-vuejs:${BUILD_NUMBER}
             docker push lovescloud/crud-mysql-vuejs:${BUILD_NUMBER}
          """
      }

      sh"""#!/bin/bash
         docker rmi lovescloud/crud-mysql-vuejs:${BUILD_NUMBER}
      """
      
   }
   

   stage('Trigger-Deploy') {
      sh label: '', script: '''sed -i \'s/IMAGE/image: lovescloud\\/crud-mysql-vuejs:\'${BUILD_NUMBER}\'/\' docker-compose.yaml
'''
      sh"""#!/bin/bash
      cat docker-compose.yaml
      kompose convert
      sudo mkdir ${BUILD_NUMBER}-kompose/
      sudo chown jenkins:jenkins ${BUILD_NUMBER}-kompose/
      sudo mv crud-mysql-vuejs-* ${BUILD_NUMBER}-kompose/
      sudo mv hk-mysql-* ${BUILD_NUMBER}-kompose/
      cd ${BUILD_NUMBER}-kompose/
      for f in * ; do mv -- "\$f" "${BUILD_NUMBER}_\$f" ; done
      cd ..
      kubectl apply -f ${BUILD_NUMBER}-kompose/

      sleep 10
      """
   }

   stage('Cleanup') {
      
      cleanWs disableDeferredWipeout: true, notFailBuild: true
   }
  
  post {
        always {
            echo 'I will always say Hello again!'
            
            emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            
        }
    }

 
  /* } catch (e) {
     // If there was an exception thrown, the build failed
     currentBuild.result = "FAILED"
     throw e
   } finally {
     // Success or failure, always send notifications
     notifyBuild(currentBuild.result)
   }
 }
 */
/* def notifyBuild(String buildStatus = 'STARTED') {
   // build status of null means successful
   buildStatus =  buildStatus ?: 'SUCCESSFUL'
 
   // Default values
   def colorName = 'RED'
   def colorCode = '#FF0000'
   def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
   def summary = "${subject} (${env.BUILD_URL})"
   def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
     <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>"""
 
   // Override default values based on build status
   if (buildStatus == 'STARTED') {
     color = 'YELLOW'
     colorCode = '#FFFF00'
   } else if (buildStatus == 'SUCCESSFUL') {
     color = 'GREEN'
     colorCode = '#00FF00'
   } else {
     color = 'RED'
     colorCode = '#FF0000'
   }
 
   // Send notifications
   //slackSend (color: colorCode, message: summary)
 
 //  hipchatSend (color: color, notify: true, message: summary)
 
   emailext (
       subject: subject,
       body: details,
       recipientProviders: [[$class: 'DevelopersRecipientProvider']]
     )
     */
 }
