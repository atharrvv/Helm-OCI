pipeline {
  agent any 

  stages {
    // stage ('docker buildd') {
    //   steps {
    //     script {
    //       docker.build('eatherv/python-application:0.0.3', '.')
    //     }
    //   }
    // }
    // stage ('docker push') {
    //   steps {
    //     script {
    //       docker.withRegistry('https://index.docker.io/v1/', 'docker') {
    //           docker.image("eatherv/python-application:0.0.3").push()
    //       }
    //     }
    //   }
    // }
    // stage ('helm package ') {
    //   steps {
    //     script {
    //       sh "helm package ./chart/application --version 3.0.0"
    //     }
    //   }
    // }
    stage ("Helm Docker Login") {
      steps {
        script {
          sh "helm registry login registry-1.docker.io -u eatherv -p dckr_pat_uxKuU-bo_5nqscqyxht2lYFfTtY"
        }
      }
    }
    //  stage ('helm push') {
    //   steps {
    //     script {
    //       sh "helm push application-3.0.0.tgz oci://registry-1.docker.io/eatherv"
    //     }
    //   }
    // }
  }
}
