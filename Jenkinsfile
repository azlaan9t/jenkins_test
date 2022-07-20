pipeline {
    agent { label 'mac' }
    environment {
        GEM_HOME="/Users/jenkins/.rvm/gems/ruby-2.7.0"
        DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer/"  //This is necessary for Fastlane to access iOS Build things.
        PATH = "/Users/jenkins/.rvm/bin:/Users/jenkins/.rvm/gems/ruby-2.7.0/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Apple/usr/bin:/Users/jenkins/Documents/flutter/bin:/Users/jenkins/Documents/flutter/.pub-cache/bin:/Users/jenkins/Documents/flutter/bin/cache/dart-sdk/bin:/Applications/Xcode.app/Contents/Developer"
    }
    stages {

        stage ('Checkout') {
            steps {
                step([$class: 'WsCleanup'])
                checkout scm
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

        stage ('Install dependencies') {
            steps {
                withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'S2A_Jenkins_ssh', keyFileVariable: 'S2A_JENKINS_PRIVATE')]) {
                    sh '''eval $(ssh-agent -s)
                            ssh-add $S2A_JENKINS_PRIVATE
                            echo '---- Install ----'
                            fvm flutter pub get
                            '''
                }
            }
        }

        stage ('Generate JSON mapping') {
            steps {
                sh "fvm flutter pub run build_runner build --delete-conflicting-outputs"
            }
        }

        stage ('Run flutter linting') {
            steps {
                sh "fvm flutter format ."
                sh "fvm flutter analyze --no-fatal-infos"
            }
        }

        stage ('Run flutter unit tests') {
            steps {
                sh "fvm flutter test"
            }
        }

        stage ('Install gems') {
            steps {
                dir('ios') {
                    sh "gem install bundler" // Install bundler in case it's not there yet
                    sh "bundle update --bundler" // In case Gemfile.lock is not created with the prev installed bundler it will update the Gemfile.lock with the installed bundler version
                    sh "bundle install" // Install all dependencies from Gemfile
                }
            }
        }

        stage('Install pods') {
            steps {
                dir('ios') {
                    sh "LANG=en_US.UTF-8 pod install --repo-update"
                }
            }
        }

        stage('Deploy') {

            parallel {

                stage('Deploy to DEV') {
                    // Deploy to dev
                    when {
                        branch 'dev'
                    }
                    steps {
                        sh "echo '---- Deployment to DEV ----'"
                        buildAndDeploy('dev')
                    }
                }

                stage('Deploy to TEST') {  
                    // Deploy to test
                    when {
                        branch 'test'
                    }
                    steps {
                        sh "echo '---- Deployment to TEST ----'"
                        buildAndDeploy('test')
                    }
                }

                stage('Deploy to DEMO') { 
                    // Deploy to demo
                    when {
                        branch 'demo'
                    }
                    steps {
                        sh "echo '---- Deployment to DEMO ----'"
                        buildAndDeploy('demo')
                    }
                }

                stage('Deploy to PROD') {  
                    // Deploy to prod
                    when {
                        branch 'prod'
                    }
                    steps {
                        sh "echo '---- Deployment to PROD ----'"
                        buildAndDeploy('prod')
                    }
                }
            }
        }
    }

    post {
        always {
            sh "fvm flutter clean"
            cleanWs()
        }
    }
}
