pipeline {
    agent any

    stages {
        stage('GIT PULL') {
            steps {
                git branch: "main", url: 'https://github.com/azlaan9t/jenkins_test.git'
            }
        }
        stage ('Flutter doctor') {
            steps {
                sh "flutter doctor -v"
            }
        }
        stage('TEST') {
            steps {
                sh "flutter test"
            }
        }
        stage('BUILD') {
            steps {
                sh '''
                  #!/bin/sh
                  flutter build apk --debug
                  '''
            }
        }
    }
}