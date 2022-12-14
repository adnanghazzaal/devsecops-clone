@Library('slack') _

pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "adnanghazzaal/numeric-app:${BUILD_ID}"
    applicationURL=   "http://devsecops-demo-adnann.eastus.cloudapp.azure.com"  
    //"http://devsecops-demo.eastus.cloudapp.azure.com"
    applicationURI="/increment/99"
  }


  stages {
      stage('Build Artifact') {
          
            steps {
              sh "printenv"
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }


  
        stage('Unit Test') {
          
            steps {
             sh "mvn test"
            }
            // post{
            //   always{
            //     junit 'target/surefire-reports/*.xml'
            //     jacoco execPattern: 'target/jacoco.exec'
            //   }
            // }
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
                     -Dsonar.host.url=$applicationURL:9000"
            }
          }
        }
        stage('SAST GATE'){
          steps{
            withSonarQubeEnv('SonarQube'){
                        timeout(time: 2, unit: 'MINUTES'){
                            waitForQualityGate abortPipeline: true
                          
                    }
             }
          }
        }
        stage('Vulnerability Scan - Docker'){
          steps{
            parallel(
              "Dependency Scan":{
                    sh "mvn dependency-check:check "
              },
              "Trivy Scan":{
                sh "bash trivy-docker-image-scan.sh"     
              },
              "OPA Conftest":{
                sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
              }         
                )
          }
          
        }
        stage('Build Docker Image and Push ') {
            steps {
             withDockerRegistry([credentialsId: "docker-hub", url: ""]){
                sh "printenv"
                sh 'sudo docker build -t adnanghazzaal/numeric-app:""$BUILD_ID"" .'
                sh 'docker push adnanghazzaal/numeric-app:""$BUILD_ID""'
              }
            }
        }   
        stage('Vulnerability Scan - Kubernetes'){
          steps{
            parallel(
            "OPA scan":{
          sh "printenv"
          sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
            },
            "Kubernetes Scan": {
              sh "bash kubesec-scan.sh"
            },
            "Trivy Scan":{
              sh "bash trivy-k8s-scan.sh"
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
            stage('Integration Tests - DEV') {
                steps {
                  script {
                    try {
                      withKubeConfig([credentialsId: 'kubeconfig']) {
                        sh "bash integration-test.sh"
                      }
                    } catch (e) {
                      withKubeConfig([credentialsId: 'kubeconfig']) {
                        sh "kubectl -n default rollout undo deploy ${deploymentName}"
                      }
                      throw e
                    }
                  }
                }
              }
        stage('OWASP ZAP - DAST') {
            steps {
          withKubeConfig([credentialsId: 'kubeconfig']){
            // we have used pipeline syntax to generate the publish HTML report below. so no need to isntall any jenkins plugin for this
            sh 'bash zap.sh'
          }
        }

      } 

      stage ("Do You Want To Push To Production"){
        steps{
          timeout(time: 2, unit: 'DAYS'){
            input 'Do You Want To Push To Prod ?'
          }
        }
      }
      stage ("Kubernetes CIS Benchmark"){
        steps{
          parallel(
            "Kube-bench Master":{
              sh "bash kube-bench-master.sh"
            },
            "kube-bench Node":{
              sh "bash kube-bench-node.sh"
            },
            "kube-bench etcd target":{

              sh "bash kube-bench-etcd.sh"
            }
          )
        }
      }
      stage('K8s Deployment -PROD'){
        steps{
          parallel(
            "Deployment":{
              withKubeConfig([credentialsId: 'kubeconfig']){
                sh "sed -i 's#replace#${imageName}#g' k8_prod-deployment_service.yaml"
                sh "kubectl -n prod apply -f k8_prod-deployment_service.yaml"
              }
            },
              "Rollout Status":{
                withKubeConfig([credentialsId: 'kubeconfig']){
                  sh "bash k8s-prod-deployment-rollout-status.sh"
                }
              }
             
          )
            }
        }
        stage('Integration Test - PROD'){
          steps{
             script{
                try{
                  withKubeConfig([credentialsId: 'kubeconfig']){
                    sh "bash integration-test-PROD.sh"
                  }
                }catch(e){
                     withKubeConfig([credentialsId: 'kubeconfig']){
                     sh "kubectl -n prod rollout undo deploy ${deploymentName}"
                     }
                  throw e
                  }
                }
              }
          
        }
      
      // stage('Testing Slack'){
      //   steps{
      //     sh 'exit 0'
      //   }
      // }
 
    }
   post{
    always{
              junit 'target/surefire-reports/*.xml'
              jacoco execPattern: 'target/jacoco.exec'
              // pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
              publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report', useWrapperFileDirectly: true])
              //Use sendNotifications.groovy from shared library and provide current build result as parameter 
          // sendNotification currentBuild.result
    }
   }
}