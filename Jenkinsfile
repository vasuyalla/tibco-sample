pipeline {
    agent any
    tools {
      maven 'maven 3.6.3'
    }
    environment {
      // Configure in global tool setting of jenkins slave or master
        //SLACK_UPLOAD_FILE_TOKEN = "dev-slack-app-token"
        CMDLINE = "clean initialize package"
        AWS_SECRET_ACCESS_KEY = "$AWS_SECRET_ACCESS_KEY_DEV"
        AWS_ACCESS_KEY_ID = "$AWS_ACCESS_KEY_ID_DEV"
        ECR_URL = "548018587351.dkr.ecr.us-east-2.amazonaws.com/testing"
        ECR_ENV_DEV = "dev"
		
        POD_PATTERN = "hello-world"
        K8S_NAMESPACE = "tibco-dev"
        DOCKER_IMAGE = "548018587351.dkr.ecr.us-east-2.amazonaws.com/testing"
    }
    stages {
      stage('Cleanup')
      {
        steps {
          jiraComment body: 'This comment was sent from Jenkins', issueKey: 'TEST-2'
          echo "Cleaning it up..."
          echo currentBuild.projectName
        }
      }
      stage('Docker Build')
      {
        steps {
          echo "Compile & Build..."
          sh 'mvn -f Helloworld.application.parent/pom.xml clean initialize package docker:build'
        }
          post {
                 always {
                     jiraSendBuildInfo site: 'devopstesting.atlassian.net', branch: 'master'
                 }
             }
      }
      stage('Docker Push')
      {
        steps {
          echo "Deploying..."
          sh '''
            export THISTAG=`date "+%y%m%d%H%M"`
            aws ecr get-login-password --region us-east-2 | docker login -u AWS --password-stdin $ECR_URL

            docker tag $DOCKER_IMAGE:latest $ECR_URL:$THISTAG
            docker push $ECR_URL:$THISTAG

            docker tag $DOCKER_IMAGE:latest $ECR_URL:$ECR_ENV_DEV
            docker push $ECR_URL:$ECR_ENV_DEV
          '''
        }
          post {
                 always {
           	     jiraSendDeploymentInfo environmentId: 'dev', environmentName: 'dev', environmentType: 'development', issueKeys: ['TEST-2']
                 }
             }
      }
      stage('JIRA') 
      {
        steps {
		 script {
		  echo "Creating Jira issue"
          testIssue = [fields: [ project: [key: 'POC'],summary: 'New JIRA Created from Jenkins.',description: 'New JIRA Created from Jenkins.',issuetype: [id: '10001']]]
          response = jiraNewIssue issue: testIssue, site: 'devopstesting.atlassian.net'
          echo response.successful.toString()
          echo response.data.toString()
          }
		  }
        }
      stage('Container Restart')
      {
        steps {
          echo "Restarting container"
          sh '''
            export PODNAME=`kubectl -n $K8S_NAMESPACE get pods | grep $POD_PATTERN | awk '{print $1}'`
            kubectl -n $K8S_NAMESPACE delete pod $PODNAME
          '''
          }
      }
      stage('QA Test')
      {
        steps {
          echo "QA Test..."
        }
      }
    }
    //options {
        //timestamps()
        //disableConcurrentBuilds()
        //buildDiscarder(logRotator(numToKeepStr: '25', artifactNumToKeepStr: '25'))
    //}
}
