def envName = "${env.JOB_NAME}".toLowerCase().replaceAll(~/[^-a-z0-9]/, '-').take(50) + "-${env.BUILD_NUMBER}"
def gitCommit = ""
def maxRunner = 1

node('master') {
  stage('Git clone') {
    deleteDir()
    checkout scm

    gitCommit = sh returnStdout: true, script: "echo -n \$(git rev-parse HEAD)"
  }

  try {
    stage('Provision new instances') {
      withCredentials([file(credentialsId: 'gcp', variable: 'gcp_key'), string(credentialsId: 'jenkin-token', variable: 'JENKINS_ACCOUNT')]) {
        sh "docker pull jmaitrehenry/google-cloud-sdk 1> /dev/null"
        sh "mkdir gcloud && cp $gcp_key gcloud/key"
        sh "docker run --rm -i -v `pwd`/gcloud:/root/.config/gcloud jmaitrehenry/google-cloud-sdk gcloud auth activate-service-account jenkins@agilequebec.iam.gserviceaccount.com --key-file=/root/.config/gcloud/key --project=agilequebec"
        sh "docker run --rm -i -v `pwd`/gcloud:/root/.config/gcloud -v `pwd`/config:/config jmaitrehenry/google-cloud-sdk gcloud deployment-manager deployments create ${envName} --template /config/gcp.jinja --properties targetSize:${maxRunner},buildId:${BUILD_NUMBER},buildName:${envName},jenkinsAccount:${JENKINS_ACCOUNT}"
      }
    }
  } finally {
    sh "docker run --rm -i -v `pwd`:/tmp jmaitrehenry/google-cloud-sdk rm -rf /tmp/gcloud || true"
  }

  try {
    stage('Run tests')  {
      node(envName) {
        git credentialsId: 'github', url: 'https://github.com/Kumojin/agiletour2018.git'

        // Build
        sh "docker build -t kumojin/agiletour2018:${gitCommit} -f Dockerfile.ci ."

        // Run
        sh "docker run kumojin/agiletour2018:${gitCommit} yarn run test:unit"
        sh "docker run kumojin/agiletour2018:${gitCommit} yarn run build"
      }
    }
  } finally {
    stage('Deallocate instances') {
      withCredentials([file(credentialsId: 'gcp', variable: 'gcp_key')]) {
        sh "docker run --rm -i -v `pwd`/gcloud:/root/.config/gcloud jmaitrehenry/google-cloud-sdk gcloud deployment-manager deployments delete -q --async ${envName}"
        sh "docker run --rm -i -v `pwd`:/tmp jmaitrehenry/google-cloud-sdk rm -rf /tmp/gcloud || true"
      }

      script {
        for (aSlave in hudson.model.Hudson.getInstance().getSlaves()) {
          if (aSlave.getLabelString() == envName) {
            aSlave.toComputer().doDoDelete();
            println('Node ' + aSlave.getNodeName() + ' deleted');
          }
        }
      }
    }
  }
}
