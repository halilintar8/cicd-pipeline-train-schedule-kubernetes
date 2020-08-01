pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        //DOCKER_IMAGE_NAME = "willbla/train-schedule"
        DOCKER_IMAGE_NAME = "halilintar8/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
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
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                        //sh "docker tag ${ORIGIN_REPO}/${REPO} ${ORIGIN_REPO}/${REPO}:${IMAGE_TAG}"
                        //sh "docker push ${ORIGIN_REPO}/${REPO}:${IMAGE_TAG}"                        
                      }
                    }
                }

                //script {
                    //docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                    //docker.withRegistry('', 'hub_docker_halilintar8') {

                        //app.push("${env.BUILD_NUMBER}")
                        //app.push("latest")

                    //}
                //}

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
        stage('DeployToProduction') {
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
        }
    }
       
}
