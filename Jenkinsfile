pipeline {
  agent any

  stages {
      stage('Build Artifact') {
         //adde comment
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
    }
}