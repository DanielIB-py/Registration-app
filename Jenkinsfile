pipeline {
    agent { label "Jenkwins-Agent" }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
            APP_NAME = "registration-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "daniel217x"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
            BRIDGE_POLARIS_APPLICATION_NAME = "WHIPSO"
            BRIDGE_POLARIS_PROJECT_NAME = "Test CI"
            BRIDGE_POLARIS_BRANCH_NAME = "main"
    }

    

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
               steps {
                   git branch: 'main', credentialsId: 'github', url: 'https://github.com/DanielIB-py/Registration-app'
               }
        }

      stage("Build Application") {
               steps {
                   sh "mvn clean package"
               }
        }

      stage("Test Application") {
               steps {
                   sh "mvn test"
               }
        }
        
    stage('polaris') {
        steps {
            withCredentials([string(credentialsId: 'poc.polarissynopsys.com', variable: 'BRIDGE_POLARIS_ACCESSTOKEN')]) {
                script {
                              
                      sh('curl -fLsS -o bridge.zip $BRIDGECLI_LINUX64 && unzip $RUNNER_TEMP bridge.zip && rm -f bridge.zip && /home/agent/workspace/Reigtration-app-ci/synopsys-bridge --verbose --stage polaris polaris.assessment.types=SAST,SCA')
                  
             
                }
            }
        }
    }
    
        stage("Build & Push Docker Image"){
            steps{
                script{
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage("Trivy Scan"){
            steps{
                script{
                    sh('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image daniel217x/register-app-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }

        stage('Cleanup Artifacts'){
            steps{
                script{
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
    }
}
