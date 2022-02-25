pipeline {
    environment {
        dockerImage = ""
    }
    agent any
    tools { 
        maven 'mvn361' 
        jdk 'jdk8' 
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }

        stage ('Code Build') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true package'
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
        }
        stage ('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build ("406971847152.dkr.ecr.us-east-1.amazonaws.com/sample-microservice" + ":${BUILD_NUMBER}", "--build-arg JARFILE=person-0.0.1-SNAPSHOT.jar .")
                }
            }
        }
        stage ('Docker Publish') {
            steps {
                 sh(label: 'ECR login and docker push', script:
         '''
            #!/bin/bash
            eval $(aws ecr get-login --region us-east-1 --no-include-email)
            # Enable Debug and Exit immediately 
            set -xe
           
            docker push 406971847152.dkr.ecr.us-east-1.amazonaws.com/sample-microservice:${BUILD_NUMBER}
         '''.stripIndent())
            }
        }
        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
    }
}
