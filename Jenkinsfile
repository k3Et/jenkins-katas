pipeline {
  agent any
  environment {
    docker_username='k3Et'
  }
  stages {
    stage('clone down') {
        agent {
            label 'host'
        }
        steps {
            stash excludes: '.git', name: 'code'
        }
    }
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
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            sh 'ls'
            deleteDir()
            sh 'ls'
          }
        }

        stage('test app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }

      }
    }

	stage('push docker app') {
	    environment {
		DOCKERCREDS = credentials('docker_login')
		}
	  steps {
		unstash 'code'
		sh 'ci/build-docker.sh'
		sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
		sh 'ci/push-docker.sh'
	  }
	  
	}

    stage('post step') {
      steps {
        deleteDir()
      }
    }

  }
}