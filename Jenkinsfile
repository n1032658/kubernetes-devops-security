pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' so that they can be downloaded later
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
        }   
    }

