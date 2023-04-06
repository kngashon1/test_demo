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
    dir('app') {
      echo 'Building Docker Image'
      script {
        echo pwd()
        echo 'Building docker from Secure Base Image'
        if (env.CURRENT_BRANCH == 'master') {
          echo 'Deploying to Fargate Master'
          withAWS(credentials: 'AWS Fargate') {
            buildDockerDeploy('example-registry.com', 'example-master-repo', 'example-master-cluster', '1.0', '<GIT_COMMIT>'.substring(0,7))
            if (env.SERVICE_TYPE != 'module') {
              restartServiceInFargate('example-master-cluster', 'example-master-service')
            } else {
              echo 'Batch job or module was deployed. No need to restart service in Fargate.'
            }
          }
        } else if (env.CURRENT_BRANCH == 'dev') {
          echo 'Deploying to Fargate Test'
          withAWS(credentials: 'AWS Fargate') {
            buildDockerDeploy('example-registry.com', 'example-stage-repo', 'example-stage-cluster', '1.0', '<GIT_COMMIT>'.substring(0,7))
            if (env.SERVICE_TYPE != 'module') {
              restartServiceInFargate('example-stage-cluster', 'example-stage-service')
            } else {
              echo 'Batch job or module was deployed. No need to restart service in Fargate.'
            }
          }
        } else {
          withAWS(credentials: 'AWS Fargate') {
            loginToAWS('example-registry.com')
            buildDockerImage('example-registry.com', 'pullrequest', '1.0')
          }
          echo 'Branch is not Dev or Master branch. Skipping deployment.'
        }
      }
    }
  }
}
 }
