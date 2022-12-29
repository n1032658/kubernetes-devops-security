pipeline {
  agent any
 environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "mdasari8019/numeric-app:${GIT_COMMIT}"
    applicationURL="http://devsecops-demo.eastus.cloudapp.azure.com"
    applicationURI="/increment/99"
  }
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }}
        
        stage('test') {
            steps {
              sh "mvn test"}
     
            }
			
			
				stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
     
    }
	
	stage('sonarqube testing') {
            steps {
             
			 withSonarQubeEnv('SonarQube') {
			 sh "mvn clean verify sonar:sonar -Dsonar.projectKey=neumeric-application -Dsonar.host.url=http://devsecops-demo.uksouth.cloudapp.azure.com:9000"
            }
			
			 timeout(time: 2, unit: 'MINUTES') {
           script {
            waitForQualityGate abortPipeline: true
         }
 }}}
			
	
	stage('Vulnerability Scan - Docker') {
      steps {
        parallel(
          "Dependency Scan": {
            sh "mvn dependency-check:check"
          },
          "Trivy Scan": {
            sh "bash Vulnerabilities_script.sh"
          },
          "OPA Conftest": {
  sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
          }
        )
      }
    }
        
			 stage('Docker Build and Push') {
       steps {
         withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
           sh 'printenv'
           sh 'sudo docker build -t mdasari8019/numeric-app:""$GIT_COMMIT"" .'
           sh 'docker push mdasari8019/numeric-app:""$GIT_COMMIT""'
         }
       }
     }
	
stage('Vulnerability Scan - Kubernetes') {
      steps {
        parallel(
          "OPA Scan": {
            sh '/usr/local/bin/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
          },
          "Kubesec Scan": {
            sh "bash kubesec-scan.sh"
          }
        )
      }
    }
	
	
	
	      stage('K8S Deployment - DEV') {
       steps {
         parallel(
           "Deployment": {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "bash k8s-deployment.sh"
             }
           },
           "Rollout Status": {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "bash k8s-deployment-rollout-status.sh"
             }
           }
         )
       }
     }
     
	 
        }

post {
always {
junit 'target/surefire-reports/*.xml'
jacoco execPattern: 'target/jacoco.exec'
#pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
}
}		
    }
