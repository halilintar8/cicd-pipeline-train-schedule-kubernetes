pipeline {
    //agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        //DOCKER_IMAGE_NAME = "willbla/train-schedule"
        DOCKER_IMAGE_NAME = "halilintar8/train-schedule"
        IMAGE_TAG = "latest"
        //BUILD_NUMBER = "0.0.1"
    }

    agent{
        node{
          label 'slave-pipeline'
        }
    }

    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Docker Build Image') {
            steps{
                container('docker') {
                    echo "Building docker image"
                    //myapp = docker.build("halilintar8/hello:latest")   
                    sh "docker build -t ${DOCKER_IMAGE_NAME} ."
                    //sh "docker build -t $tag -f jenkins-docker/Dockerfile ."
                }
            }          
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                container('docker') {
                  echo "Push docker image to hub.docker.com"
                    script {                      
                      docker.withRegistry('', 'hub_docker_halilintar8') {
                        //app.push("${env.BUILD_NUMBER}")
                        
                        //app.push("${env.DOCKER_IMAGE_NAME}")
                        //app.push("latest")

                        sh "docker tag ${DOCKER_IMAGE_NAME} ${DOCKER_IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_IMAGE_NAME}:${IMAGE_TAG}"
                      }
                    }
                }
            }
        }
        /*stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                //implement Kubernetes deployment here
            }
        }*/

        /*stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }*/

        stage('Deploy to Kubernetes') {
          steps {
            container('kubectl') {
              echo "Deploy to Kubernetes Cluster"
                script {
                  step([$class: 'KubernetesDeploy', authMethod: 'certs', apiServerUrl: 'https://10.8.1.120:6443', credentialsId:'k8sCertAuth', config: 'train-schedule-kube.yml',variableState: 'DOCKER_IMAGE_NAME,IMAGE_TAG'])
                }                
            }
          }
        }

      }
    }
       
}
