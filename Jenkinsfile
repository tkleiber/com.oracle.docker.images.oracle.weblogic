pipeline {
  agent {
    node {
      label 'docker_in_docker'
    }
  }
  options {
    buildDiscarder logRotator(numToKeepStr: '1')
  }
  stages {
    stage('Get Oracle Docker Sources') {
      steps {
        dir ('oracle') {
          git branch: 'main', credentialsId: 'github_id', url: 'https://github.com/oracle/docker-images.git'
        }
        sh 'ls -la'
      }
    }
    stage('Build Oracle Docker Images') {
      steps {
        parallel(
          "Build WebLogic": {
            dir(path: 'oracle/OracleWebLogic/dockerfiles') {
              sh 'if [ ! -f $SW_VERSION/$SW_FILE ]; then cp "$SW_DIR/$SW_FILE" $SW_VERSION/$SW_FILE; fi'
              sh 'docker pull localhost:5000/oracle/serverjre:8'
              sh 'docker tag localhost:5000/oracle/serverjre:8 oracle/serverjre:8 '
              sh './buildDockerImage.sh -v $SW_VERSION -g -s'
              sh 'docker tag oracle/weblogic:$SW_VERSION-generic localhost:5000/oracle/weblogic:$SW_VERSION-generic'
              sh 'docker push localhost:5000/oracle/weblogic:$SW_VERSION-generic'
            }
          }
        )
      }
    }
    stage('Cleanup') {
      steps {
        parallel(
          "CleanUp WebLogic": {
            sh 'docker rmi --force localhost:5000/oracle/weblogic:$SW_VERSION-generic'
            sh 'docker rmi --force oracle/weblogic:$SW_VERSION-generic'
            sh 'docker rmi --force localhost:5000/oracle/serverjre:8'
            sh 'docker rmi --force oracle/serverjre:8'
          }
        )
      }
    }
  }
  environment {
    SW_VERSION = '12.2.1.4'
    SW_FILE = 'fmw_12.2.1.4.0_wls_lite_Disk1_1of1.zip'
    SW_DIR = '/software/Oracle/WebLogic'
  }
}
