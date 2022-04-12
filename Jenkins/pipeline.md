

# Pipeline CI/CD 구성

~~~ shell
    pipeline {
          agent any

          stages {
              stage('Clone') {
                  steps {
                     git credentialsId: 'Sin_Username_Password', 
                     url: 'https://github.com/PairCompany/LogenServer.git'
                  }
              }
              stage('Build'){

                  steps {

                          sh 'pwd'
                          sh 'chmod +x ./gradlew'
                          sh './gradlew clean build'
                      }


                  post{
                      failure{
                          error 'this pipeline stops here'
                      }
                  }
              }
              stage('Deploy') {
              steps([$class: 'BapSshPromotionPublisherPlugin']) {
                  sshPublisher(
                      continueOnError: false, failOnError: true,
                      publishers:[
                          sshPublisherDesc(
                              configName: 'JHC-LOGEN',
                              verbose: true,
                              transfers: [
                                      sshTransfer(
                                          sourceFiles: "build/libs/*.war",
                                          removePrefix: "build/libs/",
                                          remoteDirectory: "/home/jenkins_home",
                                          execCommand: ""
                                          )
                                  ]
                              )
                          ]
                      )
              }
          }
          }
      }
 ~~~
