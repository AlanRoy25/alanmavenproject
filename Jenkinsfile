pipeline {
  agent {
    node {
      label 'maven-build-server'
    }
  }

  tools {
    maven 'Maven-3.9.8'
  }

  environment {
    APP_NAME = "ALAN APP"
    APP_ENV  = "PRODUCTION"
  }

  stages {
    stage('Code Build') {
      steps {
        sh 'mvn install -Dmaven.test.skip=true'
      }
    }
  
    stage('SonarQube Analysis') {
      environment {
        scannerHome = tool 'sonar-scanner'
      }
      steps {
        withSonarQubeEnv('sonarqube-server') {
          sh """
          ${scannerHome}/bin/sonar-scanner -X \
          -Dsonar.projectKey=alan-sonarorg_alan-sonarproject \
          -Dsonar.organization=alan-sonarorg \
          -Dsonar.host.url=https://sonarcloud.io \
          -Dsonar.login=1b32c75ec8c8f5b1ae7a7f0f7cc79ce67bd16d7c
          """
        }
      }
    }
    
    stage('Unit Testing Stage') {
      steps {
        sh 'mvn surefire-report:report'
      }
    }
  }
}
