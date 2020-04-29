pipeline {
  agent any
  stages {
    stage('Initialize') {
      steps {
        sh '''set +x
M2_REPOSITORY="$(./mvnw help:evaluate -Dexpression=settings.localRepository | grep .m2)"
GIT_LOCAL_BRANCH="$(git branch | sed \'s/[* ]//g\')"
GIT_LAST_COMMIT_AUTHOR="$(git log -1 --pretty=format:\'%an %ae\')"
POM_VERSION="$(cat pom.xml| grep "<version>.*</version>" | head -n 1 | awk -F\'[><]\' \'{print $3}\')"
POM_GROUPID="$(cat pom.xml| grep "<groupId>.*</groupId>" | head -n 1 | awk -F\'[><]\' \'{print $3}\')"
POM_ARTIFACTID="$(cat pom.xml| grep "<artifactId>.*</artifactId>" | head -n 1 | awk -F\'[><]\' \'{print $3}\')"
echo "$M2_HOME $M2_REPOSITORY $GIT_LOCAL_BRANCH $GIT_LAST_COMMIT_AUTHOR $POM_GROUPID $POM_ARTIFACTID $POM_VERSION"
'''
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

    stage('Test') {
      steps {
        sh '''set +x
./mvnw -B test org.owasp:dependency-check-maven:5.3.2:check sonar:sonar -pl !webgoat-integration-tests,!docker -Dformat=XML,HTML -Dmaven.test.failure.ignore=true'''
      }
    }

  }
}