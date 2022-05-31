

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
 
 
 
 ~~~shell
 pipeline {
    agent any
    
    stages {
        stage('Clone') {
            steps {
               git credentialsId: 'Sin_Username_Password',
               branch: 'deploy',
               url: 'https://github.com/PairCompany/LogenServer.git'
            }
        }
        stage('Build'){
           
            steps {
                
                    sh 'pwd'
                    sh 'cp /var/jenkins_home/workspace/properties/db.properties /var/jenkins_home/workspace/"JHC-LOGEN TEST"/src/main/resources/db.properties'
                    sh 'cp /var/jenkins_home/workspace/properties/security.properties /var/jenkins_home/workspace/"JHC-LOGEN TEST"/src/main/resources/security.properties'

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
                        configName: 'jhc-desktop',
                        verbose: true,
                        transfers: [
                                sshTransfer(
                                    sourceFiles: "build/libs/*.war",
                                    removePrefix: "build/libs/",
                                    remoteDirectory: "/home/pinpoint/deploy",
                                    execCommand: '''
                                                cd /home/pinpoint/tomcat/webapps
                                                
                                                pwd
                                            
                                                version=$(curl -s http://222.106.7.93:18080/logen/version?program=TEST03)
                                                
                                                echo "> $version"
                                                
                                                cp /home/pinpoint/deploy/logen-1.0.1-SNAPSHOT.war /home/pinpoint/tomcat/webapps/logen##$version.war
                                                
                                               
                                                
                                                
        
                                                
                                                '''
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
