pipeline {
    // The agent name must match with the jenkins node name (Manage Jenkins -> Nodes)
    agent {
        node {
            label 'maven-build-server'
        }
    }

    // The tool name must match with the jenkins tools (global configuration) variable names
    tools {
        maven 'Maven-3.9.8'
    }

    // Define environment variables (moved it to the right place)
    environment {
        APP_NAME = "Alan APP"
        APP_ENV  = "PRODUCTION"
    }

    stages {
        // SonarQube Analysis stage
        stage('Sonarqube Analysis') {
            environment {
                // Tool name must match with Jenkins Tools for Sonar Scanner - Manage Jenkins >> Tools
                scannerHome = tool 'sonar-scanner'
            }
            steps {
                // Env value must match with the Sonar Server Name - Manage Jenkins >> System
                withSonarQubeEnv('sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }
}
