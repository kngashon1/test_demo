pipeline {
	agent none  
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
}

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
stage('Prep Docker Env') {
  agent {
    label 'docker'
  }
  steps {
    echo 'DOCKER AGENT IS READY'
    echo 'Docker image will be deployed after SonarQube analysis is completed'
  }
}

stage('Prep Database Env') {
  agent {
    label 'database'
  }
  steps {
    echo 'DATABASE AGENT IS READY'
    echo 'Database image with psql will be deployed after SonarQube analysis is completed'
  }
}
stage('Update Database') {
  agent {
    label 'database'
  }
  when {
    allOf {
      expression {
        not {
          "${SERVICE_TYPE}" == 'ui'
        }
      }
      expression {
        "${UPDATE_DATABASE}" == 'true'
      }
    }
  }
  steps {
    dir('database') {
      sh '''
        for f in *.sql; do
          echo "Running ${f}"
          psql -h ${DATABASE_HOSTNAME} -p 5432 -d postgres -U postgresadmin -f "${f}"
        done
      '''
      logParser failBuildOnError: true, projectRulePath: 'parsing_rules', showGraphs: true, useProjectRule: true, parsingRulesPath: ''
      script {
        if (currentBuild.currentResult == 'FAILURE') {
          error 'Database update failed. See log for details.'
        } else {
          echo 'Database updated successfully'
        }
      }
    }
  }
}
stage('Docker Deploy to Dev') {
  agent {
    label 'docker'
  }
  steps {
    dir('app') {
      echo 'Building Docker Image'
      script {
        echo pwd()
        echo 'Building docker from Secure Base Image'
        echo 'Deploying to Fargate Test'
        withAWS(credentials: 'AWS Fargate') {
          buildDockerDeploy('demo-registry.com', 'demo-stage-repo', 'demo-stage-cluster', '1.0', '<GIT_COMMIT>'.substring(0,7))
          if (env.SERVICE_TYPE != 'module') {
            restartServiceInFargate('demo-stage-cluster', 'demo-stage-service')
          } else {
            echo 'Batch job or module was deployed. No need to restart service in Fargate.'
          }
        }
      }
    }
  }
}
}