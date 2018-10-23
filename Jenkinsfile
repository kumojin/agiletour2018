def gitCommit = ""

node('master') {
  stage('Git clone') {
    git credentialsId: 'github', url: 'https://github.com/Kumojin/agiletour2018.git'

    gitCommit = sh returnStdout: true, script: "echo -n \$(git rev-parse HEAD)"
  }
  stage('Run tests')  {
      // Build
      sh "docker build -t kumojin/agiletour2018:${gitCommit} -f Dockerfile.ci ."

      // Run
      sh "docker run kumojin/agiletour2018:${gitCommit} yarn run test:unit"
      sh "docker run kumojin/agiletour2018:${gitCommit} yarn run build"
  }
}