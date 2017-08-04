pipeline {
  agent {
    node {
      label 'localhost_vagrant'
    }

  }
  stages {
    stage('Get Oracle Docker Sources') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: 'origin/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false, timeout: 30]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'e7677693-d5e6-463f-b6d4-597e0fb91c3f', url: 'https://github.com/oracle/docker-images.git']]])
      }
    }
    stage('Build Oracle Docker Images') {
      steps {
        parallel(
          "Build WebLogic": {
            dir(path: 'OracleWebLogic/dockerfiles') {
              sh 'if [ ! -f $SW_VERSION/$SW_FILE ]; then cp "$SW_DIR/$SW_FILE" $SW_VERSION/$SW_FILE; fi'
              sh 'sudo ./buildDockerImage.sh -v $SW_VERSION -g'
              sh 'docker tag oracle/weblogic:$SW_VERSION-generic localhost:5000/oracle/weblogic:$SW_VERSION-generic'
              sh 'docker push localhost:5000/oracle/weblogic$SW_VERSION-generic'
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
          }
        )
      }
    }
  }
  environment {
    SW_VERSION = '12.2.1.2'
    SW_FILE = 'fmw_12.2.1.2.0_wls_Disk1_1of1.zip'
    SW_DIR = '/software/Oracle/WebLogic'
  }
}
