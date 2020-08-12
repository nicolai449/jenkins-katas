pipeline {
  agent any
  stages {
    stage('Clone Down') {
      steps {
        stash(excludes: '.git', name: 'code')
      }
    }

    stage('Parallel Execution') {
      parallel {
        stage('Say Hello') {
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
            skipDefaultCheckout true
            stash(excludes: '.git', name: 'code')
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

    stage('build docker app') {
      steps {
        unstash 'code'
        sh 'ci/build-docker.sh'
        stash(excludes: '.git', name: 'code')
      }
    }

    stage('Master branch build') {
      when {
        branch 'master'
      }
      environment {
        DOCKERCREDS = credentials('docker_login')
      }
      steps {
        unstash 'code'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u" $DOCKERCREDS_USR" --password-stdin'
        sh 'ci/push-docker.sh'
        stash(excludes: '.git', name: 'code')
      }
    }

    stage('Component test') {
      when {
        branch pattern: '/dev', comparator: 'GLOB'
      }
      steps {
        unstash 'code'
        sh 'ci/component-test.sh'
      }
    }

  }
  environment {
    docker_username = 'nissero'
  }
}