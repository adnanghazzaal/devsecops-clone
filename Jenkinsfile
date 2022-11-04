pipeline {
  agent any

  stages {
      stage('Build Artifact') {
          
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }  
        stage('Unit Test') {
          
            steps {
             sh "mvn test"
            }
        }  
        //commit
        stage('Build Docker Image and Push to Docker Hub') {
            steps {
             withDockerRegistry([credentialsId: "docker-hub", url: ""]){
                sh "printenv"
                sh 'docker build -t adnanghazzaal/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push adnanghazzaal/numeric-app:""GIT_COMMIT""'
              }
            }
        }    
    }
}