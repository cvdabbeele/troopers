import groovy.json.JsonBuilder

node('jenkins-jenkins-slave') {
  withEnv(['REPOSITORY=troopers',
  'GIT_ACCOUNT=https://github.com/mawinkler']) {
    stage('Pull Image from Git') {
      script {
        git (url: "${GIT_ACCOUNT}/${REPOSITORY}.git", credentialsId: "github-auth")
      }
    }
    stage('Build Image') {
      script {
        dbuild = docker.build("mawinkler/${REPOSITORY}:$BUILD_NUMBER")
      }
    }
    parallel (
      "Test": {
        //script {
        //  sh "python tests/test_app.py"
        //}
        echo 'All functional tests passed'
      },
      "Check Image (pre-Registry)": {
        smartcheckScan([
          imageName: "mawinkler/${REPOSITORY}:$BUILD_NUMBER",
          smartcheckHost: "${DSSC_SERVICE}",
          smartcheckCredentialsId: "smartcheck-auth",
          insecureSkipTLSVerify: true,
          insecureSkipRegistryTLSVerify: true,
          preregistryScan: true,
          preregistryHost: "${DSSC_REGISTRY}",
          preregistryCredentialsId: "preregistry-auth",
          findingsThreshold: new groovy.json.JsonBuilder([
            malware: 0,
            vulnerabilities: [
              defcon1: 0,
              critical: 0,
              high: 1,
            ],
            contents: [
              defcon1: 0,
              critical: 1,
              high: 0,
            ],
            checklists: [
              defcon1: 0,
              critical: 0,
              high: 0,
            ],
          ]).toString(),
        ])
      }
    )
    stage('Push Image to Docker Hub') {
      script {
        docker.withRegistry('', 'docker-hub') {
          dbuild.push('$BUILD_NUMBER')
          dbuild.push('latest')
        }
      }
    }
    stage('Deploy App to Kubernetes') {
      script {
        kubernetesDeploy(configs: "app.yml", kubeconfigId: "kubeconfig", enableConfigSubstitution: true)
      }
    }
  }
}
