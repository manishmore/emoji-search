pipeline {
    environment {
      DOCKER = credentials('docker-hub')
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
            sh 'docker run --name nodeapp-dev --network="bridge" -d \
            -p 9000:9000 nodeapp-dev'
            sh 'docker run --name test-image -v $PWD:/JUnit --network="bridge" \
            --link=nodeapp-dev -d -p 3010:3000 \
            test-image:latest'
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
                            sh 'docker tag nodeapp-dev:trunk <DockerHub Username>/nodeapp-prod:latest'
                            sh 'docker push <DockerHub Username>/nodeapp-prod:latest'
                            sh 'docker save <DockerHub Username>/nodeapp-prod:latest | gzip > nodeapp-prod-golden.tar.gz'
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
        junit 'reports.xml'
        archiveArtifacts(artifacts: 'reports.xml', allowEmptyArchive: true)
        archiveArtifacts(artifacts: 'nodeapp-prod-golden.tar.gz', allowEmptyArchive: true)
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
