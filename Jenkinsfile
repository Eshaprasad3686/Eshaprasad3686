pipeline{
    agent any
  /*  tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
  */
    environment{
        registry = 'thundereagle36/vprofileapp'
        registryCredentials = 'dockerhub'
    }
    stages{

        stage('Build'){
            steps{
                sh 'mvn install -DskipTest'
            }

            post{
                success{
                    echo "archiving it ..."
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('Unit Test'){
            steps{
                sh 'mvn test'
            }
        }
        stage("Integration Test"){
            steps{
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('checkstyle Analysis'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
            post{
                success{
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage('Sonar Analysis'){
            environment{
                scannerHome = tool 'sonar4.7'
            }
            steps{
                withSonarQubeEnv('sonar-pro'){
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                         -Dsonar.projectName=vprofile \
                         -Dsonar.projectVersion=1.0 \
                         -Dsonar.sources=src/ \
                         -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                         -Dsonar.junit.reportsPath=target/surefire-reports/ \
                         -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                         -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage('Quality Gate'){
            steps{
                timeout(time: 1, unit: 'HOURS'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build App Image'){
            steps{
                script{
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                }
            }
        }
        stage('Upload Image'){
            steps{
                script{
                    docker.withRegistry('',registryCredentials){
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Remove Unused Image'){
            steps{
                sh 'docker rmi $registry:V$BUILD_NUMBER'
            }
        }

        stage('Kubernetes Deploy'){
            agent{label 'KOPS'}
            steps{
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
            }
        }
    }

}