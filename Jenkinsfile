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
        REGISTRY_URL = 'https://binhost.jfrog.io'
    }

    stages {
        stage('Code Build') {
            steps {
                echo 'Building the project...'
                sh 'mvn install -Dmaven.test.skip=true'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonar-scanner'
            }
            steps {
                echo 'Performing SonarQube Analysis...'
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                    ${scannerHome}/bin/sonar-scanner \
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
                echo 'Running Unit Tests...'
                sh 'mvn surefire-report:report'
            }
        }

        stage('Jar Publish') {
            steps {
                script {
                    echo '<--------------- Started Publishing Jar --------------->'

                    // Verify that the directory exists
                    if (!fileExists('jarstaging')) {
                        error 'Directory "jarstaging/" does not exist or contains no .jar files.'
                    }

                    // Set up Artifactory server
                    def server = Artifactory.newServer(
                        url: "${REGISTRY_URL}/artifactory",
                        credentialsId: "artifactory_token"
                    )

                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "libs-release-local/{1}",
                                "flat": false,
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""

                    // Log the uploadSpec
                    echo "Upload Specification: ${uploadSpec}"

                    // Retry mechanism for upload
                    retry(3) {
                        def buildInfo = server.upload(uploadSpec)
                        buildInfo.env.collect()
                        server.publishBuildInfo(buildInfo)
                    }

                    echo '<--------------- Jar Published Successfully --------------->'
                }
            }
        }
    }
}
