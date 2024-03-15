pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: gradle
spec:
  containers:
  - name: gradle
    image: gradle:6.3.0-jdk11
    command:
    - cat
    tty: true
    env:
      # Define the environment variable
      - name: CRED
        valueFrom:
          secretKeyRef:
            name: jenkinscred
            key: ECR_CREDENTIAL_JSON
  restartPolicy: Never
"""      
    }
  }
  environment {
    IMAGE_REGISTRY = "339713125195.dkr.ecr.us-east-1.amazonaws.com"
  }
  stages {

    stage('Approval') {
      when {
        branch 'main'
      }
      steps {
        script {
          def plan = 'cartservice CI'
          input message: "Do you want to build and push?",
              parameters: [text(name: 'Plan', description: 'Please review the work', defaultValue: plan)]
        }
      } 
    }

    stage('build and push docker image') {
      when {
        branch 'main'
      }            
      steps {
        container('gradle') {
          sh 'gradle jib --no-daemon --image ${IMAGE_REGISTRY}/eshop-cartservice:latest -Djib.to.auth.username=AWS -Djib.to.auth.password=$CRED'  
        }
      }
      post {
        success { 
          slackSend(channel: 'C06MTJQJA05', color: 'good', message: "(Job : ${env.JOB_NAME} - Build Number : ${env.BUILD_NUMBER}) CI success - from <@U06NMQU5K36>")
        }
        failure {
          slackSend(channel: 'C06MTJQJA05', color: 'danger', message: "(Job : ${env.JOB_NAME} - Build Number : ${env.BUILD_NUMBER}) CI fail - from <@U06NMQU5K36>")
        }
      }
    }
  }
}