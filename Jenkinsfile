pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "siddharth67/numeric-app:${GIT_COMMIT}"
    applicationURL="http://devsecops-demo.eastus.cloudapp.azure.com"
    applicationURI="/increment/99"
  }

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
            post { 
              always { 
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
        } 
    }
 } 
      stage('Mutation Tests - PIT') {
            steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post { 
              always { 
                pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          }
        }  
      }  

     
      stage('SonarQube - SAST') {
            steps {
              withSonarQubeEnv('SonarQube') {
                sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops12.eastus.cloudapp.azure.com:9000"
              }
              timeout(time: 2, unit: 'MINUTES') {
                script {
                  waitForQualityGate abortPipeline: true
                }
              }
            }
          }

    // stage('Vulnerability Scan - Docker') {
    //       steps {
    //           sh "mvn dependency-check:check"
    //           }
    //       post { 
    //         always { 
    //           dependencyCheckPublisher pattern: '**/target/dependency-check-report.xml'
    //     }
    //   }  
    // }
         stage('Vulnerability Scan - Docker') {
              steps {
                parallel(
                  "Dependency Scan": {
                    sh "mvn dependency-check:check"
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
          // When creating credentials in Jenkins for Docker Hub, use a username other than your email address; otherwise, an error will occur when jenkins try to push the image.
           stage('Docker Build and Push') {
                steps {
                  withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                    sh 'printenv'
                    sh 'sudo docker build -t gags23deep/numeric-app:""$GIT_COMMIT"" .'
                    sh 'docker push gags23deep/numeric-app:""$GIT_COMMIT""'
                  }
                }
              }
            
            stage('Vulnerability Scan - Kubernetes') {
                  steps {
                    parallel(
                      "OPA Scan": {
                        sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
                      },
                      // "Kubesec Scan": {
                      //   sh "bash kubesec-scan.sh"
                      // },
                      // "Trivy Scan": {
                      //   sh "bash trivy-k8s-scan.sh"
                      // }
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
                    // "Rollout Status": {
                    //   withKubeConfig([credentialsId: 'kubeconfig']) {
                    //     sh "bash k8s-deployment-rollout-status.sh"
                    //   }
                    // }
                  )
                }
              }
                  
  }

}

