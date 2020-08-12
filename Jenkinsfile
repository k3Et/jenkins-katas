pipeline {
  agent any
  stages {
    stage('Parallel execution') {
      parallel {
        stage('build') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          steps {
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            sh 'ls'
            deleteDir()
            sh 'ls'
          }
        }

        stage('test app') {
          steps {
            sh 'ci/unit-test-app.sh'
            sh 'junit \'app/build/test-results/test/TEST-*.xml\''
          }
        }

      }
    }

    stage('post step') {
      steps {
        deleteDir()
      }
    }

  }
}