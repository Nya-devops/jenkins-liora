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
                    echo "BRANCH_NAME is: ${env.BRANCH_NAME}"
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
                    echo 'Deploying to DEV environment...'
                    sh "kubectl apply -f secrets-movie.yaml -n ${NAMESPACE_DEV}"
                    sh "kubectl apply -f secrets-cast.yaml -n ${NAMESPACE_DEV}"
                    sh "kubectl apply -f statefulset-moviedb.yaml -n ${NAMESPACE_DEV}"
                    sh "kubectl apply -f statefulset-castdb.yaml -n ${NAMESPACE_DEV}"

                    sh "helm upgrade --install movie-service ./charts -f values-movie.yaml -n ${NAMESPACE_DEV}"
                    sh "helm upgrade --install cast-service ./charts -f values-cast.yaml -n ${NAMESPACE_DEV}"
                }
            }
        }

        stage("Deploy to QA"){
            steps{
                script{
                    echo 'Deploying to QA environment...'
                    sh "kubectl apply -f secrets-movie.yaml -n ${NAMESPACE_QA}"
                    sh "kubectl apply -f secrets-cast.yaml -n ${NAMESPACE_QA}"
                    sh "kubectl apply -f statefulset-moviedb.yaml -n ${NAMESPACE_QA}"
                    sh "kubectl apply -f statefulset-castdb.yaml -n ${NAMESPACE_QA}"

                    sh "helm upgrade --install movie-service ./charts -f values-movie.yaml -n ${NAMESPACE_QA}"
                    sh "helm upgrade --install cast-service ./charts -f values-cast.yaml -n ${NAMESPACE_QA}"
                }
            }
        }

        stage("Deploy to STAGING"){
            steps{
                script{
                    echo 'Deploying to STAGING environment...'
                    sh "kubectl apply -f secrets-movie.yaml -n ${NAMESPACE_STAGING}"
                    sh "kubectl apply -f secrets-cast.yaml -n ${NAMESPACE_STAGING}"
                    sh "kubectl apply -f statefulset-moviedb.yaml -n ${NAMESPACE_STAGING}"
                    sh "kubectl apply -f statefulset-castdb.yaml -n ${NAMESPACE_STAGING}"

                    sh "helm upgrade --install movie-service ./charts -f values-movie.yaml -n ${NAMESPACE_STAGING}"
                    sh "helm upgrade --install cast-service ./charts -f values-cast.yaml -n ${NAMESPACE_STAGING}"
                }
            }
        }
        
        stage("Deploy to PROD"){
            when{
                expression { env.GIT_BRANCH == 'origin/main'}
            }
            input{
                message "Press PROCEED to continue deployment to Production" 
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
    }

    post {
        always {
            echo 'Pipeline finished, thanks for your input !'
        }       
    }
}