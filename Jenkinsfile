// anchore plugin for jenkins: https://www.jenkins.io/doc/pipeline/steps/anchore-container-scanner/

pipeline {
  environment {
    // "registry" isn't required if we're using docker hub, I'm leaving it here in case you want to use a different registry
    // registry = 'registry.hub.docker.com'
    // you need a credential named 'docker-hub' with your DockerID/password to push images
    registryCredential = 'docker-hub'
    DOCKER_HUB = credentials(${registryCredential})
    // change this repository and imageLine to your DockerID
    repository = "$DOCKER_HUB_USER}/jenkins-plugin-bug"
    tag = ":testcase1-${BUILD_NUMBER}"
    imageLine = "pvnovarese/jenkins-plugin-bug:testcase1-${BUILD_NUMBER} Dockerfile"
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
          dockerImage = docker.build repository + tag
          docker.withRegistry( '', registryCredential ) { 
            dockerImage.push() 
          }
        }
      }
    }
    stage('Analyze with Anchore plugin') {
      steps {
        writeFile file: 'anchore_images', text: imageLine
        script {
          try {
            anchore name: 'anchore_images', forceAnalyze: 'true', engineRetries: '900'
          } catch (err) {
            // if scan fails, clean up (delete the image) and fail the build
            sh 'docker rmi $repository$tag'
            sh 'exit 1'
          }
        }
        // forceAnalyze is required since we're passing a Dockerfile with the image
      }
    }
    stage('Re-tag as prod and push stable image to registry') {
      steps {
        script {
          docker.withRegistry('', registryCredential) {
            dockerImage.push('prod') 
            // dockerImage.push takes the argument as a new tag for the image before pushing
          }
        }
      }
    }
    stage('Clean up') {
      // if we succuessfully pushed the :prod tag than we don't need the $BUILD_ID tag anymore
      steps {
        sh 'docker rmi $repository$tag $repository:prod'
      }
    }
  }
}
