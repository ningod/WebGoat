#!groovy

pipeline {
  agent any
  stages {
    stage('Initialize') {
      steps {
        script {
          M2_REPOSITORY=getM2LocalRepository()
          GIT_LOCAL_BRANCH="$(git branch | sed \'s/[* ]//g\')"
          GIT_LAST_COMMIT_AUTHOR="$(git log -1 --pretty=format:\'%an %ae\')"
          POM_VERSION=getPomVersion()
          POM_GROUPID=getPomGroupId()
          POM_ARTIFACTID=getPomArtifactId()
          echo "$M2_HOME $M2_REPOSITORY $GIT_LOCAL_BRANCH $GIT_LAST_COMMIT_AUTHOR $POM_GROUPID $POM_ARTIFACTID $POM_VERSION"
        }
    }

    stage('Build') {
      steps {
        sh '''
          set +x
          ./mvnw clean compile
        '''
        sh '''
          set +x
          #FIXME Could not find artifact org.owasp.webgoat:webgoat-container:jar:tests:v8.0.0-SNAPSHOT
          ./mvnw install -B -f ./webgoat-container/pom.xml && ./mvnw -B install -Dmaven.test.skip=true
         '''
      }
    }
    stage('Sonarqube') {
        environment {
            scannerHome = tool 'sonarqube-scanner-4.3'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
            timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }
    stage('Unit Test') {
      steps {
        sh '''set +x
          ./mvnw -B test org.owasp:dependency-check-maven:5.3.2:check sonar:sonar -pl !webgoat-integration-tests,!docker -Dformat=XML,HTML -Dmaven.test.failure.ignore=true'''
      }
    }

  }
}

def getShortCommitHash() {
    return sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
}

def getChangeAuthorName() {
    return sh(returnStdout: true, script: "git show -s --pretty=%an").trim()
}

def getChangeAuthorEmail() {
    return sh(returnStdout: true, script: "git show -s --pretty=%ae").trim()
}

def getChangeSet() {
    return sh(returnStdout: true, script: 'git diff-tree --no-commit-id --name-status -r HEAD').trim()
}

def getChangeLog() {
    return sh(returnStdout: true, script: "git log --date=short --pretty=format:'%ad %aN <%ae> %n%n%x09* %s%d%n%b'").trim()
}

def getCurrentBranch () {
    return sh (
            script: 'git rev-parse --abbrev-ref HEAD',
            returnStdout: true
    ).trim()
}



def getM2LocalRepository () {
    return sh (
            script: './mvnw help:evaluate -Dexpression=settings.localRepository | grep .m2',
            returnStdout: true
    ).trim()
}

def getPomGroupId () {
    return sh (
            script: 'cat pom.xml| grep "<groupId>.*</groupId>" | head -n 1 | awk -F\'[><]\' \'{print $3}\'',
            returnStdout: true
    ).trim()
}

def getPomArtifactId () {
    return sh (
            script: 'cat pom.xml| grep "<artifactId>.*</artifactId>" | head -n 1 | awk -F\'[><]\' \'{print $3}\'',
            returnStdout: true
    ).trim()
}

def getPomVersion () {
    return sh (
            script: 'cat pom.xml| grep "<version>.*</version>" | head -n 1 | awk -F\'[><]\' \'{print $3}\'',
            returnStdout: true
    ).trim()
}
