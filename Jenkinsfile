#!/usr/bin/env groovy

/* Only keep the 5 most recent builds. */
def projectProperties = [
    [$class: 'BuildDiscarderProperty',strategy: [$class: 'LogRotator', numToKeepStr: '5']],
]

if (!env.CHANGE_ID) {
    if (env.BRANCH_NAME == null) {
        projectProperties.add(pipelineTriggers([cron('H/30 * * * *')]))
    }
}

properties(projectProperties)

try {
    /* Assuming that wherever we're going to build, we have nodes labelled with
    * "Docker" so we can have our own isolated build environment
    */
    node {
      if (env.NODE_NAME == "master") {
        currentBuild.displayName = "DEV"
        currentBuild.description = "Development stages"
      }
      else {
        currentBuild.displayName = "TAP"
        currentBuild.description = "Integration-UAT-Production stages"
      }
      stage ("Build") {
        echo "Kick-off MVN Build"
        sh 'mvn --version'
      }
      stage ("Deploy") {
        echo "Do an actual TIBCO deployment"
        withCredentials([usernamePassword(credentialsId: '7780a8e4-da7d-4144-9636-30609c19f025', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          // available as an env variable, but will be masked if you try to print it out any which way
          sh 'echo $PASSWORD'
          // also available as a Groovy variable—note double quotes for string interpolation
          echo "$USERNAME"
        }
        withCredentials([string(credentialsId: '123abc', variable: 'mySecretText')]) {
          sh 'echo $mySecretText > /tmp/mysecret'
        }
      }
      stage ("Test") {
        if (currentBuild.displayName == "DEV") {
          echo "Start unit-tests with SOAPui"
        }
      } 
      stage ("Release") {
        echo "Store possible shippable product in the Nexus Repository"
      }
    }

}
catch (exc) {
    echo "Caught: ${exc}"

    String recipient = 'infra@lists.jenkins-ci.org'

    mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}) failed",
            body: "It appears that ${env.BUILD_URL} is failing, somebody should do something about that",
              to: recipient,
         replyTo: recipient,
            from: 'noreply@ci.jenkins.io'

    /* Rethrow to fail the Pipeline properly */
    throw exc
}

// vim: ft=groovy
