pipeline {
    agent any
    environment {
        DOCKERHUB_USER = 'nyamzz' 
        NAMESPACE_DEV = 'dev'
        NAMESPACE_QA = 'qa'
        NAMESPACE_STAGING = 'staging'
        NAMESPACE_PROD = 'prod'
    }

    stages {
        stage("Build and Push"){
            steps{
                script{
                    echo 'building the docker images...'
                    sh "docker build -t ${DOCKERHUB_USER}/movie-service:1.0.0 ./movie-service"
                    sh "docker build -t ${DOCKERHUB_USER}/cast-service:1.0.0 ./cast-service"

                    echo 'pushing the docker images...'
                    withCredentials([usernamePassword(credentialsId: 'docker-token', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]){
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${DOCKERHUB_USER}/movie-service:1.0.0"
                        sh "docker push ${DOCKERHUB_USER}/cast-service:1.0.0"
                    }
                }
            }
        }

        stage("Deploy to DEV"){
            steps{
                script{
                    echo 'deploying to DEV environment...'
                    sh 'kubectl apply -f secrets-movie.yaml -n dev'
                    sh 'kubectl apply -f secrets-cast.yaml -n dev'
                    sh 'kubectl apply -f statefulset-moviedb.yaml -n dev'
                    sh 'kubectl apply -f statefulset-castdb.yaml -n dev'

                    sh 'helm upgrade --install movie-service ./charts -f values-movie.yaml -n dev'
                    sh 'helm upgrade --install cast-service ./charts -f values-cast.yaml -n dev'
                }
            }
        }

        stage("Deploy to PROD"){
            when{
                branch 'main'
            }
            input{
                message "Press OK to continue deployment to Production" 
            }   
            steps{
                script{
                    echo 'Ok: Deploying to PROD environment...'
                    sh "kubectl apply -f secrets-movie.yaml -n ${NAMESPACE_PROD}"
                    sh "kubectl apply -f secrets-cast.yaml -n ${NAMESPACE_PROD}"
                    sh "kubectl apply -f statefulset-moviedb.yaml -n ${NAMESPACE_PROD}"
                    sh "kubectl apply -f statefulset-castdb.yaml -n ${NAMESPACE_PROD}"

                    sh "helm upgrade --install movie-service ./charts -f values-movie.yaml -n ${NAMESPACE_PROD}"
                    sh "helm upgrade --install cast-service ./charts -f values-cast.yaml -n ${NAMESPACE_PROD}"
                }
            }
        }

    post{
        always{
            echo 'Pipeline finished !'
            }       
        }
    }
}