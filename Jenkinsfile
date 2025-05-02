pipeline {
  agent any 

  stages {
    stage ('docker buildd') {
      steps {
        script {
          docker.build('eatherv/python-application:0.0.3', '.')
        }
      }
    }
    stage ('docker push') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'docker') {
              docker.image("eatherv/python-application:0.0.3").push()
          }
        }
      }
    }
    stage ('helm package ') {
      steps {
        script {
          sh "helm package ./chart/application --version 3.0.0"
        }
      }
    }
   //   stage ('helm push') {
   //    steps {
   //      script {
          
   //      }
   //    }
   // }
  
  }
}
