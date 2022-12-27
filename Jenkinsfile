pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }}
        
        stage('test') {
            steps {
              sh "mvn test"}
              post {
always {
junit 'target/surefire-reports/*.xml'
jacoco execPattern: 'target/jacoco.exec'
}
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
	 
	      stage('K8S Deployment - DEV') {
       steps {
        
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "sed -i 's#replace#$mdasari8019/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
               sh "kubectl  apply -f k8s_deployment_service.yaml"
             }
           }
          
        
       }
     
	 
        }   
    }

