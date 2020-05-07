pipeline {

    agent none

    environment {

        NODE_ENV="homolog"
        AWS_ACCESS_KEY=""
        AWS_SECRET_ACCESS_KEY=""
        AWS_SDK_LOAD_CONFIG="0"
        BUCKET_NAME="app-digital"
        REGION="us-east-1" 
        PERMISSION=""
        ACCEPTED_FILE_FORMATS_ARRAY=""
        VERSION="1.0.0"
    }


    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    triggers {
        cron('@daily')
    }

    stages{
        stage("Build, Test and Push Docker Image") {
            agent {  
                node {
                    label 'master'
                }
            }
            stages {

                stage('Clone repository') {
                    steps {
                        script {
                            if(env.GIT_BRANCH=='origin/homolog'){
                                checkout scm
                            }
                            sh('printenv | sort')
                            echo "My branch is: ${env.GIT_BRANCH}"
                        }
                    }
                }
                stage('Build image'){       
                    steps {
                        script {
                            print "Environment will be : ${env.NODE_ENV}"
                            docker.build("devops:latest")
                        }
                    }
                }

                stage('Test image') {
                    steps {
                        script {

                            docker.image("devops:latest").withRun('-p 8030:3000') { c ->
                                sh 'docker ps'
                                sh 'sleep 10'
                                sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'
                                
                            }
                    
                        }
                    }
                }

                stage('Docker push') {
                    steps {
                        echo 'Push latest para AWS ECR'
                        script {
                            docker.withRegistry('https://402303658132.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:awskey') {
                                docker.image('devops').push()
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Homolog') {
            agent {  
                node {
                    label 'homolog'
                }
            }

            steps { 
                script {
                    if(env.GIT_BRANCH=='origin/homolog'){
 
                        docker.withRegistry('https://402303658132.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:awskey') {
                            docker.image('devops').pull()
                        }

                        echo 'Deploy para Desenvolvimento'
                        sh "hostname"
                        // sh "docker stop app1"
                        // sh "docker rm app1"
                        //sh "docker run -d --name app1 -p 8030:3000 933273154934.dkr.ecr.us-east-1.amazonaws.com/digitalhouse-devops:latest"
                        withCredentials([[$class:'AmazonWebServicesCredentialsBinding' 
                            , credentialsId: 'homologs3']]) {
                        sh "docker run -d --name app1 -p 8030:3000 -e NODE_ENV=homolog -e AWS_ACCESS_KEY=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY -e BUCKET_NAME=digitalhouse-grupo2-homolog 402303658132.dkr.ecr.us-east-1.amazonaws.com/devops:latest"
                        }
                        
                        sh "docker ps"
                        sh 'sleep 10'
                        sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'

                    }
                }
            }

        }

        stage('Deploy to Producao') {
            agent {  
                node {
                    label 'producao'
                }
            }

            steps { 
                script {
                    if(env.GIT_BRANCH=='origin/producao'){
 
                        environment {

                            NODE_ENV="production"
                            AWS_ACCESS_KEY="123456"
                            AWS_SECRET_ACCESS_KEY="asdfghjkkll"
                            AWS_SDK_LOAD_CONFIG="0"
                            BUCKET_NAME="app-digital"
                            REGION="us-east-1" 
                            PERMISSION=""
                            ACCEPTED_FILE_FORMATS_ARRAY=""
                        }


                        docker.withRegistry('https://402303658132.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:awskey') {
                            docker.image('devops').pull()
                        }

                        echo 'Deploy para Producao'
                        sh "hostname"
                        // sh "docker stop app1"
                        // sh "docker rm app1"
                        //sh "docker run -d --name app1 -p 8030:3000 933273154934.dkr.ecr.us-east-1.amazonaws.com/digitalhouse-devops:latest"
                        withCredentials([[$class:'AmazonWebServicesCredentialsBinding' 
                            , credentialsId: 'prods3']]) {
                          sh "docker run -d --name app1 -p 8030:3000 -e NODE_ENV=producao -e AWS_ACCESS_KEY=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY -e BUCKET_NAME=digitalhouse-grupo2-producao 402303658132.dkr.ecr.us-east-1.amazonaws.com/devops:latest"
                        }
                        sh "docker ps"
                        sh 'sleep 10'
                        sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'

                    }
                }
            }

        }

    }
}
