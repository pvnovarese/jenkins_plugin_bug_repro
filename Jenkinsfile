// anchore plugin for jenkins: https://www.jenkins.io/doc/pipeline/steps/anchore-container-scanner/

pipeline {
  environment {
    // "REGISTRY" isn't required if we're using docker hub, I'm leaving it here in case you want to use a different registry
    // REGISTRY = 'registry.hub.docker.com'
    // you need a credential named 'docker-hub' with your DockerID/password to push images
    CREDENTIAL = "docker-hub"
    DOCKER_HUB = credentials("$CREDENTIAL")
    REPOSITORY = "${DOCKER_HUB_USR}/jenkins-plugin-bug"
    TAG = ":testcase1-${BUILD_NUMBER}"
    IMAGELINE = "${REPOSITORY}${TAG} Dockerfile"
  }
  agent any
  stages {
    stage('Checkout SCM') {
      steps {
        checkout scm
      }
    }
    stage('Build image and push to registry') {
      steps {
        script {
          dockerImage = docker.build REPOSITORY + TAG
          docker.withRegistry( '', CREDENTIAL ) { 
            dockerImage.push() 
          }
        }
      }
    }
    stage('Analyze with Anchore plugin') {
      steps {
        writeFile file: 'anchore_images', text: IMAGELINE
        script {
          try {
            // forceAnalyze is a good idea since we're passing a Dockerfile with the image
            anchore name: 'anchore_images', forceAnalyze: 'true', engineRetries: '900'
          } catch (err) {
            // if scan fails, clean up (delete the image) and fail the build
            sh 'docker rmi ${REPOSITORY}${TAG}'
            sh 'exit 1'
          }
        }
      }
    }
    stage('Re-tag as prod and push stable image to registry') {
      steps {
        script {
          docker.withRegistry('', CREDENTIAL) {
            dockerImage.push('prod') 
            // dockerImage.push takes the argument as a new tag for the image before pushing
          }
        }
      }
    }
    stage('Clean up') {
      // get rid of both the pre-scan tag and prod tag
      steps {
        sh 'docker rmi ${REPOSITORY}${TAG} ${REPOSITORY}:prod'
      }
    }
  }
}
