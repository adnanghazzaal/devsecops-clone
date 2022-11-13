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
                  sh "mvn clean verify sonar:sonar \
                  -Dsonar.projectKey=numeric-application \
                  -Dsonar.host.url=http://devsecops-demo-adnan.eastus.cloudapp.azure.com:9000"
            withSonarQubeEnv('SonarQube'){
                        timeout(time: 2, unit: 'MINUTES'){
                          script{
                            waitForQualityGate abortPipeline: true
                        }    
                    }
             }

            // sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-demo-adnan.eastus.cloudapp.azure.com:9000 \
            //     -Dsonar.login=sqp_9285466cdd966311b4d4438a97fc58343417dcd4"
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