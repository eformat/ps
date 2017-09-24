#!/usr/bin/groovy
def versionPrefix = ""
try {
  versionPrefix = VERSION_PREFIX
} catch (Throwable e) {
  versionPrefix = "1.0"
}

def canaryVersion = "${versionPrefix}.${env.BUILD_NUMBER}"

def utils = new io.fabric8.Utils()
def label = "buildpod.${env.JOB_NAME}.${env.BUILD_NUMBER}".replace('-', '_').replace('/', '_')

mavenNode{
  def envStage = utils.environmentNamespace('petstore-staging')
  def envProd = utils.environmentNamespace('petstore-prod')
  
  git = git branch: 'master', credentialsId: 'eformat', url: 'https://github.com/eformat/ps'

  container(name: 'maven') {

    stage 'Build Release'
    mavenCanaryRelease {
      version = canaryVersion
    }

    stage 'Rollout Production'
    kubernetesApply(environment: envProd)

  }
}
