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
        stage('Build Docker Image and Push ') {
            steps {
             withDockerRegistry([credentialsId: "docker-hub", url: ""]){
                sh "printenv"
                sh 'docker build -t adnanghazzaal/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push adnanghazzaal/numeric-app:""$GIT_COMMIT""'
              }
            }
        }   
        stage('Kubernete Deployment - DEV') {
            steps {
             withKubeConfig([credentialsId: kubeconfig]) {
              sh "sed -i 's/replace/adnanghazzaal/numeric-app:${GIT_COMMIT}/g' k8s_deployment_service.yml"
              sh "kubectl apply -f k8s_deployment_service.yml"
             }
            }
        }  
    }
}