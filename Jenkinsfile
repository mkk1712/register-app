pipeline {
    agent any
    stages {
        stage("Checkout from SCM"){
                steps {
		    echo 'checking out code from Git...'	
                    git branch: 'main', url: 'https://github.com/mkk1712/register-app'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

       stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'sonarqub') { 
                        sh "mvn admin:admin"
		        }
	           }	
           }
       }

       stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

       }

       stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ashfaque9x/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }

       stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-232-128-192.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
       }
    }

    post {
       failure {
             emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                      subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
                      mimeType: 'text/html',to: "maheshreddy0421@gmail.com"
      }
      success {
            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                     subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
                     mimeType: 'text/html',to: "maheshreddy0421@gmail.com"
      }      
   }
}
