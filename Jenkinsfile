pipeline {
    agent any
    tools {
        go 'Go 1.24.3'
    }
  environment {
        SONAR_TOKEN = credentials('sonar-token') // Jenkins credential ID
        SONAR_HOST_URL = 'https://68308f6c8c1b3edc072bf40a-c4bff7.node-ap-a1de.iximiuz.com'
        SONAR_SCANER_HOME= tool 'SonarQube'
        GOCACHE = "${WORKSPACE}/.go-cache"
    }
    stages {
        stage('Git Checkout') {
            steps {
                cleanWs()
                git branch: 'develop', url: 'https://github.com/Shopping-App-Services/shopping-service.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                     sh '''
                    ${SONAR_SCANER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=ShippingService \
                        -Dsonar.sources=. \
                        -Dsonar.language=go \
                        -Dsonar.java.binaries=. \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.verbose=true
                    '''
                    }  
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Test') {
            steps {
              
                sh 'go test ./...'
            }
        }
         stage('Build') {
            steps {
                sh 'go version'
                sh 'go build -o ./...'
                
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                    // This step should not normally be used in your script. Consult the inline help for details.
                  withDockerRegistry(credentialsId: '20226572-5e4c-4db0-ad81-6762251b3d09', toolName: 'docker') {
                        sh 'ls -latr'
                        sh "docker build -t shipping-service ."
                        sh "docker tag shipping-service nitesh2611/shipping-service:latest "
                        sh "docker push nitesh2611/shipping-service:latest "
                    }
                }
            }
        }
    }
}


