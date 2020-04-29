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
        sh '''#FIXME Could not find artifact org.owasp.webgoat:webgoat-container:jar:tests:v8.0.0-SNAPSHOT
./mvnw install -pl !webgoat-integration-tests -Dmaven.test.skip=true

'''
      }
    }

    stage('Test') {
      parallel {
        stage('Unit Test') {
          steps {
            sh './mvnw test -pl !webgoat-integration-tests,!docker -Dmaven.test.failure.ignore=true'
            junit(allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml')
          }
        }

        stage('SAST') {
          steps {
            withSonarQubeEnv(installationName: 'sonarqube-scanner-4.3', credentialsId: 'sonarqube') {
              sh './mvnw org.owasp:dependency-check-maven:5.3.2:check sonar:sonar -pl !webgoat-integration-tests,!docker -Dformat=XML,HTML -Dmaven.test.skip=true'
              waitForQualityGate(credentialsId: 'sonarqube', abortPipeline: true)
            }

          }
        }

      }
    }

  }
}