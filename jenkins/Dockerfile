FROM jenkins/jenkins:lt
COPY executors.groovy /usr/share/jenkins/ref/init.groovy.d/executors.groovy
RUN /usr/local/bin/install-plugins.sh docker-slaves github-branch-source:1.8

