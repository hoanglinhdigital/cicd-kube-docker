pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        ARTVERSION = "${env.BUILD_ID}"
        registry="linhnv26/vprofileapp"
        registryCredential= "dockerhub"

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

        stage('Build Docker App image'){
            steps{
                script{
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                }
            }
        }
        stage('Upload Image'){
            steps{
                script{
                    docker.withRegistry('', registryCredential){
                        docker.Image.push(":V$BUILD_NUMBER")
                        docker.Image.push('latest')
                    }
                }
            }
        }

        stage('Remove docker images'){
            steps{
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }

        stage('Kubernetes deploy'){
            steps{
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
            }
        }
    }


}
