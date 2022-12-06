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
                pitmutation mutationStatFile: '**/target/pit-reports/**/mutations.xml'
                     }
                  }  
      }         
  }
}