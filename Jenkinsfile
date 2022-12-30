@Library('slack') _
pipeline {
  agent any
 environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "mdasari8019/numeric-app:${GIT_COMMIT}"
    applicationURL="http://devsecops-demo.uksouth.cloudapp.azure.com"
    applicationURI="/increment/99"
  }
  stages {
    
	  
	    stage('Prompte to PROD?') {
       steps {
         timeout(time: 2, unit: 'DAYS') {
           input 'Do you want to Approve the Deployment to Production Environment/Namespace?'
         }
       }
     }

     stage('K8S CIS Benchmark') {
       steps {
         script {

           parallel(
             "Master": {
               sh "bash cis-master.sh"
             },
             "Etcd": {
               sh "bash cis-etcd.sh"
             },
             "Kubelet": {
               sh "bash cis-kubelet.sh"
             }
           )

         }
       }
	   }
        }

post {
always {
//junit 'target/surefire-reports/*.xml'
//jacoco execPattern: 'target/jacoco.exec'
// pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
//dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
//publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Reports', reportTitles: 'OWASP ZAP HTML Reports'])
    // Use sendNotifications.groovy from shared library and provide current build result as parameter 
     sendNotification currentBuild.result
}
}		
    }
