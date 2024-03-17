pipeline {
    agent any
    environment {
        DOCKER_HUB_USER_NAME="ahmedatiia11"
        DOCKER_IMAGE_NAME = "${DOCKER_HUB_USER_NAME}/netfilx-clone"
        // TMDB_V3_API_KEY="8720293778a54d900600819481d26434"
    }

    stages {
      //Cloning CI Repo to git the last version of the code
      stage('Clone repository') {  
        steps {
        checkout scm
    }
      }
      // Build Dockerfile and Rename it with the Dockerhub-reg Account name and Add the commmit number as tag name 
      stage('Test and Build Docker Image') {
         steps {       
         script { withCredentials([string(credentialsId: 'TMDB_V3_API_KEY', variable: 'tmdb')]) {
                    env.GIT_COMMIT_REV = sh (script: 'git log -n 1 --pretty=format:"%h"', returnStdout: true)
                    sh "docker build --build-arg TMDB_V3_API_KEY=${tmdb} -t ${DOCKER_IMAGE_NAME}:${GIT_COMMIT_REV} ."
                    }
                  }
              }                           
     }
     // push Docker Image to the registry with the modified name 
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', "docker-cred" ) {
            sh 'docker push ${DOCKER_IMAGE_NAME}:${GIT_COMMIT_REV}'
          }
        }
      }
    }

    stage("TRIVY Unit Test"){
            steps{
                sh "trivy --version" 
            }
        }
    stage("TRIVY Scanner"){
            steps{
                sh "trivy image ${DOCKER_IMAGE_NAME}:${GIT_COMMIT_REV} > trivy-image-${GIT_COMMIT_REV}.txt" 
            }
        }
        
    // Trigger  the CD job at Jenkins
        stage('Trigger CD job ') {
                steps {
                echo "triggering CD"
                build job: 'Netfilx-clone-CD', parameters: [string(name: 'GIT_COMMIT_REV', value: env.GIT_COMMIT_REV)]
        }
        }      
  }

}
