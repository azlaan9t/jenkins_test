pipeline {
    agent any

    stages {
        stage('GIT PULL') {
            steps {
                git branch: "main", url: 'https://github.com/azlaan9t/jenkins_test.git'
            }
        }
        stage ('Setting up the FVM') {
            steps {
                sh "fvm install 3.0.1"
                sh "fvm use 3.0.1"
                sh "fvm flutter precache"
            }
        }
        stage ('Flutter doctor') {
            steps {
                sh "fvm flutter doctor -v"
            }
        }
        stage('TEST') {
            steps {
                sh "fvm flutter test"
            }
        }
        stage('BUILD') {
            steps {
                sh '''
                  #!/bin/sh
                  fvm flutter build apk --debug
                  '''
                sh '''
                  #!/bin/sh
                  fvm flutter build ios --profile
                  '''
            }
        }
    }
}