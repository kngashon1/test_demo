pipeline {
	agent none 
environment
		{	
			//If no custom directory, leave as '.'
			//SERVICE_TYPE should be set to 'api', 'module', or 'ui'
			//POSTMAN_READY and UPDATE_DATABASE are toggles for Api Tests stage and Update Database stage, respectively
			
			//variables
			WORKING_DIRECTORY = '.'
			MASTER_ECR_REPO = 'stage-pipeline-test'
			DEV_ECR_REPO = 'dev-pipeline-test'
			MASTER_SERVICE_NAME = 'stage-pipeline-test' 
			DEV_SERVICE_NAME = 'dev-pipeline-test'
			MAVEN_BUILD_COMMAND = 'mvn -U -Dmaven.repo.local=repo clean install test'
			SONARQUBE_PROJECT_NAME = 'test-pipeline'
			POSTMAN_READY = 'false'
			UPDATE_DATABASE = 'false'
			SERVICE_TYPE = 'api'	
			REPORT_NAME = 'test' 
           }
	
    stages {

        stage('Initialize') {
  agent {
    label 'default'
  }
  steps {
    echo "PATH = ${PATH}"
    echo "M2_HOME = ${M2_HOME}"
    echo "Current branch = <CURRENT_BRANCH>"
    echo "Building version <BUILD_VERSION>"
    echo "Bitbucket CommitID = <GIT_COMMIT>".substring(0,7)
  }
}
/*
     stage('Docker Backup') {
  agent {
    label 'default'
  }
  steps {
    dir(WORKING_DIRECTORY) {
      echo 'Saving a backup copy of existing Docker image from Fargate'
      withAWS(credentials: 'AWS Fargate') {
        backupDocker(REGISTRY_URL, DEV_ECR_REPO)
      }
    }
  }
}*/

  	stage('Build and Test') {

     parallel {

    stage('Maven Build') {
              agent {
               label 'default'
          }

    steps {
           dir('app') {
          echo 'Building in Maven'
          sh 'mvn clean install'
        }
      }
      post {
        always {
          junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: false
          archiveArtifacts artifacts: '**/target/surefire-reports/*.xml', fingerprint: true
          archiveArtifacts artifacts: '**/target/site/*-surefire-report.html', fingerprint: true
        }
      }
    }
  }
}
}

stage('Docker Deploy') {
  agent {
    label 'docker'
  }
   steps {
              script{
                        docker.withRegistry('https://720766170633.dkr.ecr.us-east-2.amazonaws.com', 'ecr:us-east-2:aws-credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                    }
                }
            }
        }
    }
