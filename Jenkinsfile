pipeline {
  agent none
  stages {
    stage('Maven Install') {
      agent {
        docker {
          image 'maven:3.5.0'
          args '-v /var/.m2:/root/.m2'
        }
      }
      steps {
        sh 'mvn clean install -DskipTests=true'
      }
    }

    stage('Unit test') {
      agent {
        docker {
          image 'maven:3.5.0'
          args '-v /var/.m2:/root/.m2'
        }
      }
      steps {
        sh 'mvn test'
      }
    }

    stage('SonarQube Analysis') {
      agent {
        docker {
          image 'maven:amazoncorretto'
          args '-v /var/.m2:/root/.m2'
        }
      }
      steps {
        withSonarQubeEnv(installationName: 'SonarPetclinic', credentialsId: 'sonar-petclinic-key') {
          sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.8.0.2131:sonar'
        }
      }
    }

    stage('Quality Gate'){
          steps{
            timeout(time:1, unit:'HOURS'){
              waitForQualityGate abortPipeline: true
            }
          }
        }

    stage('Docker Build') {
    agent any
      steps {
        sh 'docker build -t grupo07/spring-petclinic:latest .'
      }
    }
  }
  post {
    always {
        node('master'){
            sh 'find . -name "TEST-*.xml" -exec touch {} \\;'
            junit(testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true)
        }
    }
  }
}
