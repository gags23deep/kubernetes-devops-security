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
                sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops12.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_48dc806251ce3b34d4780d316a0e5cf1c3a0e804"
              }
              timeout(time: 2, unit: 'MINUTES') {
                script {
                  waitForQualityGate abortPipeline: true
                }
              }
            }
          }

//       stage('Vulnerability Scan - Docker') {
//             steps {
//                 sh "mvn dependency-check:check"
//                 }
//             post { 
//               always { 
//                 dependencyCheckPublisher pattern: '/target/dependency-check-report.xml'
//           }
//         }  
//       }
          
      
        
  }

}

