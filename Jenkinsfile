node {
  def registry = "containers.schibsted.io"
  def serviceName = "spt-nextgen-vgnext/bff"
  def deploymentId = "${BUILD_NUMBER}"

  //sh "mkdir ${HOME}/.docker"
  //sh "cp /secrets/.dockerconfigjson ${HOME}/.docker/config.json"

  stage("Checkout") {
    checkout scm
  }

  def imageTag = getGitCommit();

  docker.withRegistry(registry) {
    runTests(serviceName, "tests", "")
    buildService(serviceName, imageTag)

    deploy(serviceName, registry, imageTag, deploymentId)
  }
}

def runTests(serviceName, target = "test", extraArgs) {
    def testImage = null

    stage("Build tests") {
        testImage = docker.build(serviceName + "-tests", "--pull .")
        testImage.push()
    }

    stage("Run ${target}") {
        testImage.inside() {
            sh "npm run ${target} ${extraArgs}"
        }
    }
}

def buildService(serviceName, imageTag) {
    stage("Build service") {
        def serviceImage = docker.build(serviceName, "--pull --build-arg NODE_ENV=production .")
        serviceImage.push(imageTag) // Pushes versioned image
        serviceImage.push() // Pushes latests
    }
}

def deploy(serviceName, registry, imageTag, deploymentId) {
    stage("Deploy") {
        sh """cat k8s.yml | sed -r \
            -e 's@(image:).+@\\1 ${registry}/${serviceName}:${imageTag}@' \
            -e 's@(build:).+@\\1 ${imageTag}@' \
            -e 's@(deployment:).+@\\1 v${deploymentId}@' | \
            tee k8s-deployed.yml
            """
            //kubectl apply -f -
    }
}

def getGitCommit() {
    gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    return gitCommit.take(6)
}
