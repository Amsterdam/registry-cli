def tryStep(String message, Closure block, Closure tearDown = null ) {
    try {
        block()
    }
    catch (Throwable t) {
        slackSend message: "${env.JOB_NAME}: ${message} failure ${env.BUILD_URL}", channel: '#ci-channel', color: 'danger'

        // stop container
        if(Container) {
          Container.stop
        }

        throw t

    }
    finally {
        if (tearDown) {
            tearDown()
        }
    }
}

String BRANCH = "${env.BRANCH_NAME}"

node {
    // define objects
    def Image
    def Container
    def commitHash = checkout(scm).GIT_COMMIT
    def shortCommit = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()

    stage("Checkout") {
        checkout scm
    }

    stage("Build image") {
        tryStep "build", {
          Image = docker.build("amsterdam/registry-cli:${shortCommit}", ".")
        }
    }

    if (BRANCH == "master") {
      stage("Push image") {
          tryStep "Push", {
            withCredentials([usernamePassword( credentialsId: 'docker-hub-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
              docker.withRegistry('', 'docker-hub-credentials') {
                sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                Image.push()
                Image.push('production')
                Image.push('latest')
              }
            }
          }
      }
    }
}
