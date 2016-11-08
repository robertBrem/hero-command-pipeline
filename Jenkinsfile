withEnv(["KUBERNETES_HOST_NAME=hero-command-test"]) {
  stage "checkout, build, test and publish"
  node {
    git poll: true, url: "http://adesso.disruptor.ninja:30130/rob/hero-command.git"
    def mvnHome = tool 'M3'
    sh "${mvnHome}/bin/mvn clean install"
    sh "USER_NAME=robertbrem VERSION=1.0.${currentBuild.number} ./build.js"
    step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
  }

  stage "systemtest"
  node {
    git poll: true, url: "https://github.com/robertBrem/hero-command-st.git"
    def mvnHome = tool 'M3'
    sh "./stop.js"
    sh "VERSION=1.0.${currentBuild.number} ./run.js"
    sh "${mvnHome}/bin/mvn clean install failsafe:integration-test"
    step([$class: 'JUnitResultArchiver', testResults: '**/target/failsafe-reports/TEST-*.xml'])
  }

  stage "ui/smoke test"
  node {
    git poll: true, url: "https://github.com/robertBrem/hero-command-ut.git"
    def mvnHome = tool 'M3'
    sh "${mvnHome}/bin/mvn clean install failsafe:integration-test"
    step([$class: 'JUnitResultArchiver', testResults: '**/target/failsafe-reports/TEST-*.xml'])
  }
  
  stage "lasttest"
  node {
    git poll: true, url: "https://github.com/robertBrem/hero-command-lt.git"
    def mvnHome = tool 'M3'
    sh "${mvnHome}/bin/mvn clean verify -Dperformancetest.webservice.host=adesso.disruptor.ninja -Dperformancetest.webservice.port=31080"
  }

  stage "consumer driven contract test"
  node {
    git poll: true, url: "https://github.com/robertBrem/hero-command-cdct.git"
    def mvnHome = tool 'M3'
    sh "${mvnHome}/bin/mvn clean install failsafe:integration-test"
    step([$class: 'JUnitResultArchiver', testResults: '**/target/failsafe-reports/TEST-*.xml'])
  }

  stage ("Manuel tests on http://adesso.disruptor.ninja:31080/hero-command/resources/heros")
    input 'Everything ok?'
    node {
  }

  stage "canary"
  input "Deploy a canary with the new version?"
  node {
    git poll: true, url: "https://github.com/robertBrem/hero-command-canary.git"
    sh "VERSION=1.0.${currentBuild.number} ./run.js"
  }

  stage "full prod"
  input "Undeploy older version"
  node {
    git poll: true, url: "https://github.com/robertBrem/hero-command-prod.git"
    sh "VERSION=1.0.${currentBuild.number} ./run.js"
  }

}
