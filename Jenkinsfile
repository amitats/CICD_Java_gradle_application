pipeline{
    agent any
    environment{
    VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh 'java -version'
                        sh './gradlew sonarqube'
                      }  

                      timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                  }     
                }  
            }
            stage("docker build & docker push"){
            steps{
                script{

                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                               sh '''
                                docker build -t 18.184.159.46:8083/springapp:${VERSION} .
                                docker login -u admin -p $docker_password 18.184.159.46:8083 
                                docker push 18.184.159.46:8083/springapp:${VERSION}
                                docker rmi 18.184.159.46:8083/springapp:${VERSION}
                              '''
                            }
                         }
                    }
                }
             
           stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                          dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$docker_password http://18.184.159.46:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                       }
                   }
               }
            }
          }
       }

          

    


        

     
    