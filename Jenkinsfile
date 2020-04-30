#!groovy

pipeline {
  
  agent any
  
  stages {
    
    stage('Initialize') {
      
         
      steps {
        sh "printenv | sort"
        script {
          M2_REPOSITORY=getM2LocalRepository()          
          GIT_LAST_COMMIT_AUTHOR=getLastCommitAuthor()
          POM_VERSION=getPomVersion()
          POM_GROUPID=getPomGroupId()
          POM_ARTIFACTID=getPomArtifactId()
          echo "${env.M2_HOME} ${M2_REPOSITORY} ${env.GIT_LOCAL_BRANCH} ${GIT_LAST_COMMIT_AUTHOR} ${POM_GROUPID} ${POM_ARTIFACTID} ${POM_VERSION}"
        }
      }
    }

    stage('Build') {
      
      agent {
          docker {
              image 'docker.io/openjdk:11-jdk'
              label 'javabuild-11-jdk'
              args  '-v maven_repository:/root/.m2/repository'            
              reuseNode: true
          }
      }//End Agent      
      
      steps {
        script {
          echo "compile parent pom ${POM_GROUPID} ${POM_ARTIFACTID} ${POM_VERSION}"
          sh './mvnw -q -B clean compile'
          echo "package parent pom ${POM_GROUPID} ${POM_ARTIFACTID} ${POM_VERSION}"
          sh './mvnw -q -B install -f ./webgoat-container/pom.xml && ./mvnw -B install -Dmaven.test.skip=true'
        }
      }
    }
 
      
    stage('Unit Test') {

      
      steps {
        script {
          echo "test parent pom ${POM_GROUPID} ${POM_ARTIFACTID} ${POM_VERSION}"
          sh './mvnw -q -B test -pl !webgoat-integration-tests,!docker -Dmaven.test.failure.ignore=true'
          junit allowEmptyResults: true, testResults: '**/target/surefire-reports/**/*.xml' 
         }
      }

      
    }//End Stage Unit Test
      
    stage('SAST') {
      
      tools {
        jdk "jdk8"
      }
           
      steps {
        
        withSonarQubeEnv('owasp/sonarqube') {
        
          script {
              echo "org.owasp:dependency-check-maven:5.3.2:check parent pom ${POM_GROUPID} ${POM_ARTIFACTID} ${POM_VERSION}"
              sh './mvnw -B -q org.owasp:dependency-check-maven:5.3.2:aggregate -Dformats=XML,HTML -Dmaven.test.skip=true'
              echo "org.cyclonedx:cyclonedx-maven-plugin:1.6.4 parent pom ${POM_GROUPID} ${POM_ARTIFACTID} ${POM_VERSION}"
              sh "./mvnw org.cyclonedx:cyclonedx-maven-plugin:1.6.4:makeAggregateBom"
          }          

          script {
              echo "sonar:sonar parent pom ${POM_GROUPID} ${POM_ARTIFACTID} ${POM_VERSION}"
              sh './mvnw -B -q sonar:sonar -pl !webgoat-integration-tests,!docker -Dmaven.test.skip=true'
          }    

          script {
              echo "dependencytrack upload projectName ${POM_ARTIFACTID}"
              dependencyTrackPublisher (
                artifact: "target/dependency-check-report.xml",
                artifactType: "Dependency-Check Scan Result (XML)",
                projectId: "fdd9377f-a007-479d-93ef-92e98a8479cd", 
                projectName: "${POM_ARTIFACTID}",
                projectVersion: "${POM_VERSION}",
                synchronous: true            
              )
          }

          timeout(time: 10, unit: 'MINUTES') {
                  waitForQualityGate abortPipeline: true
          }  
          
        }// End WithSonarQubeEnv
      }//End SAST steps
    }//ENd SAST Stage

  }//End Stages
    
post {
        /*
         * These steps will run at the end of the pipeline based on the condition.
         * Post conditions run in order regardless of their place in pipeline
         * 1. always - always run
         * 2. changed - run if something changed from last run
         * 3. aborted, success, unstable or failure - depending on status
         */
        always {
            echo "I AM ALWAYS first"            
        }
        aborted {
            echo "BUILD ABORTED"
        }
        success {
            echo "BUILD SUCCESS"
            echo "Keep Current Build If branch is master"
        }
        unstable {
            echo "BUILD UNSTABLE"
        }
        failure {
            echo "BUILD FAILURE"
        }
    }// End Post
}//End Pipeline


def getLastCommitAuthor() {
  return sh(returnStdout: true, script: "git log -1 --pretty=format:\'%an %ae\'").trim()
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

