pipeline {
  agent {
    docker {
      image 'openjdk:11-jdk'
      args '--expose 8080 -e PIPELINE_NETWORK -e VIRTUAL_HOST=webgoat.integration.${PIPELINE_NETWORK}'
    }

  }
  stages {
    stage('Initialize') {
      steps {
        sh '''export GIT_LOCAL_BRANCH="$(git branch | sed \'s/[* ]//g\')"
export GIT_LAST_COMMIT_AUTHOR="$(git log -1 --pretty=format:\'%an %ae\')"
export POM_VERSION="$(cat pom.xml| grep "<version>.*</version>" | head -n 1 | awk -F\'[><]\' \'{print $3}\')"
export POM_GROUPID="$(cat pom.xml| grep "<groupId>.*</groupId>" | head -n 1 | awk -F\'[><]\' \'{print $3}\')"
export POM_ARTIFACTID="$(cat pom.xml| grep "<artifactId>.*</artifactId>" | head -n 1 | awk -F\'[><]\' \'{print $3}\')"'''
      }
    }

    stage('Build') {
      steps {
        sh './mvnw clean compile'
        sh './mvnw package -Dmaven.test.skip=true\' '
      }
    }

  }
}