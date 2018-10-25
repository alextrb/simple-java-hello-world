// Configuration des images docker utilisées pour les différentes étapes CI/CD
pipeline {
  agent {
    kubernetes {
      label 'mypod'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: some-label-value
spec:
  containers:
  - name: maven
    image: maven:alpine
    command:
    - cat
    tty: true
  - name: docker
    image: docker
    command:
    - cat
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
    }
  }


// Configuration à modifier
  stages {

    stage('Build') {
      steps {
        container('maven') {
          sh 'mvn -B -DskipTests clean package'
        }
      }
    }

    stage('Test') {
      steps {
          /* `make check` returns non-zero on test failures,
          * using `true` to allow the Pipeline to continue nonetheless
          */
          container('maven') {
            sh 'mvn test'
          }
      }
    }

    stage('Build docker') {
      steps {
        /* `make check` returns non-zero on test failures,
        * using `true` to allow the Pipeline to continue nonetheless
        */
        container('docker') {
          sh 'docker build -t my-app:$BUILD_NUMBER'
        }
      }
    }

    stage('Run') {
      steps {
        /* `make check` returns non-zero on test failures,
        * using `true` to allow the Pipeline to continue nonetheless
        */
        container('docker') {
          sh 'docker run my-app:$BUILD_NUMBER'
        }
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/*.xml'
    }
  }
}