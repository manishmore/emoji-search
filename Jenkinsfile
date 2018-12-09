pipeline {
    environment {
    // env
    }
  agent any
  stages {
// Building your Test Images
    stage('BUILD') {
      parallel {
        stage('create network') {
          steps {
            sh 'docker network create boilerplate'
          }
        }
        stage('Emoji Search') {
          steps {
            sh 'docker-compose up -d'
          }
        }
        stage('Test-Unit Image') {
          steps {
            sh 'docker run -it -p 3003:3000 emoji-search_frontend &'
          }
        }
      }
      post {
        failure {
            echo 'This build has failed. See logs for details.'
        }
      }
    }
// Performing Software Tests
    stage('TEST') {
      parallel {
        stage('Mocha Tests') {
          steps {
            sh 'docker run --name nodeapp-dev --network="boilerplate" -d \
            -p 3009:3000 manishmore14/emoji-search:1.0'
          }
        }
        stage('Quality Tests') {
          steps {
            echo "Quality Tests"
          }
        }
      }
      post {
        success {
            echo 'Build succeeded.'
        }
        unstable {
            echo 'This build returned an unstable status.'
        }
        failure {
            echo 'This build has failed. See logs for details.'
        }
      }
    }
// Deploying your Software
    stage('DEPLOY') {
          when {
           branch 'master'  //only run these steps on the master branch
          }
            steps {
                    retry(3) {
                        timeout(time:10, unit: 'MINUTES') {
                         echo "unit TEST"
                        }
                    }

            }
            post {
                failure {
                    sh 'docker stop nodeapp-dev test-image'
                    sh 'docker image prune -a'
                    sh 'docker container prune --filter "until=1h"'
                    deleteDir()
                }
            }
    }
// JUnit reports and artifacts saving
    stage('REPORTS') {
      steps {
         echo "REPORTS"
      }
    }
// Doing containers clean-up to avoid conflicts in future builds
    stage('CLEAN-UP') {
      steps {
        sh 'docker stop nodeapp-dev test-image'
        sh 'docker image prune -a'
        sh 'docker container prune --filter "until=1h"'
        deleteDir()
      }
    }
  }
}
