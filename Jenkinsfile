

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ( 'build App image') {
          steps {
            script {
               dockerImage = docker.build registry + ":V$BUILD_NUMBER
            }

          }
        }
        stage("upload Image"){
          steps {
            script {
              docker.withRegistry('', registryCredential) {
                docker.Image.push ("V$BUILD_NUMBER")
                dockerImage.push("VBUILD_NUMBER")
                dockerImage.push('latest')'
              }
            }
          }

        }
        stage('remove unused docker image'){
          steps{
            sh "docker rmi $registry:V$BUILD_NUMBER"

          }
        }
         stage('kubernetes Deploy') {
           steps {
             sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${VBUILD_NUMBER} --Nnamespace s3blaise."
           }
         }




    }


}
