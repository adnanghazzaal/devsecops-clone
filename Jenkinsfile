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
            post{
              always{
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }  
        stage ('Mutation Tests- PIT'){
          steps{
            sh 'mvn org.pitest:pitest-maven:mutationCoverage'
          }
          post{
            always{
              pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            }
          }
        }
        stage('SAST SonarQube'){
          steps{
            withSonarQubeEnv('SonarQube'){
            sh "mvn clean verify sonar:sonar \
                  -Dsonar.projectKey=numeric-application \
                  -Dsonar.host.url=http://devsecops-demo-adnan.eastus.cloudapp.azure.com:9000"
                  // timeout(time: 20, unit: 'MINUTES'){
                  //   scrpit{ 
                  //     waitForQualityGate abortPipeline: true
                  //   }
                  // }
                  }
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
             withKubeConfig([credentialsId: "kubeconfig"]) {
              sh "sed -i 's#replace#adnanghazzaal/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
              sh "kubectl apply -f k8s_deployment_service.yaml"
             }
            }
        }  
    }
}