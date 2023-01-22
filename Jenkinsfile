pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
       registry = "bdino2005/dinoappdoker"
       registryCredential = "dockerhub1"
        ARTVERSION = "${env.BUILD_ID}"
    }

    stages{


        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ( 'build App image') {
          steps {
            script {
               dockerImage = docker.build registry + ":V$BUILD_NUMBER"
            }

          }
        }
        stage("upload Image"){
          steps {
            script {
              docker.withRegistry('', registryCredential) {
                docker.Image.push ("V$BUILD_NUMBER")
                dockerImage.push("VBUILD_NUMBER")
                dockerImage.push('latest')
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
