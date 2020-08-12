pipeline {
  agent any
  environment {
    docker_username='k3et'
  }
  stages {
    stage('clone down') {
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
          when {
            beforeAgent true
            branch 'master'
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash 'build'
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
	  when { 
      branch "master" 
      }
    environment {
		  DOCKERCREDS = credentials('docker_login')
		}
	  steps {
		  unstash 'build'
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
    stage('Master branch build') {
      when { 
        not {
          branch "dev/*"    
        }
      }
        steps {
          unstash 'code'
          sh 'ci/component-test.sh'
        }
    }
  }
}