pipeline {
  agent any 

  environment {
    CHART_NAME = "application"
    CHART_VERSION = "5.0.0"
    OCI_REGISTRY = "oci://registry-1.docker.io/eatherv"
  }

  stages {
    stage ('docker buildd') {
      steps {
        script {
          docker.build('eatherv/python-application:0.0.2', '.')
        }
      }
    }
    stage ('docker push') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'docker') {
              docker.image("eatherv/python-application:0.0.2").push()
          }
        }
      }
    }
    stage ('helm package ') {
      steps {
        script {
          sh "helm package ./chart/application --version ${CHART_VERSION}"
        }
      }
    }
    stage ("Helm Docker Login") {
      steps {
        script {
          withCredentials([usernamePassword(
                    credentialsId: 'docker',  // Update with your Jenkins credential ID
                    usernameVariable: 'USER',
                    passwordVariable: 'TOKEN'
                )]) {
            sh """ helm registry login registry-1.docker.io -u ${USER} -p ${TOKEN} """
            }
          // sh "helm registry login registry-1.docker.io -u eatherv -p"
        }
      }
    }
     stage ('helm push') {
      steps {
        script {
          sh "helm push ${CHART_NAME}-${CHART_VERSION}.tgz ${OCI_REGISTRY}"
        }
      }
    }
  }
}
