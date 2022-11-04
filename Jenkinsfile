pipeline {
  agent any

  stages {
      stage('Build Artifact') {
          
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
              added 
            }
        }  
        stage('Unit Test') {
          
            steps {
             sh "mvn test"
            }
        }  
        stage('Build Docker Image and Push to Docker Hub') {
            steps {
             sh "printenv"
             sh 'docker build -t adnanghazzaal/numeric-app:""$GIT_COMMIT"" .'
             sh 'docker push adnanghazzaal/numeric-app:""GIT_COMMIT""'
            }
        }    
    }
}