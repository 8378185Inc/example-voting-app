pipeline {

    agent none
      
    stages {
        stage('worker-build') {
          when{
            changeset "**/worker/**"
          }
        agent {
          docker {
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }

            steps {
                echo 'Building worker app'
                dir('worker') {
                  sh 'mvn compile'
                }
            }
        }
        stage('worker-test') {
          when{
            changeset "**/worker/**"
          }
        agent {
          docker {
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }

            steps {
                echo 'Running Unit Tests on worker app...'
                dir('worker'){
                  sh 'mvn clean test'
                }
            }
        }
        stage("worker-package") {
          when{
            changeset "**/worker/**"
          }
        agent {
          docker {
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }
            steps{
              echo 'Packaging worker app into a jarfile'
              dir('worker'){
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
              }
            }
          }

        stage('worker-docker-package'){
          agent any
          when{
            changeset "**/worker/**"
          }
          steps{
            echo 'Packaging worker app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/','dockerlogin') 
              {
                def workerImage = docker.build("rotimi98/worker:v${env.BUILD_ID}", "./worker")
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
                workerImage.push("latest")
              }
            }
          }
        }
        stage('result-build') {

          when{
            changeset "**/result/**"
          }
        agent{
          docker {
            image 'node:8.16.0-alpine'
          }
        }

            steps {
                echo 'Compiling result app...'
                dir('result') {
                  sh 'npm install'
                }
            }
        }
        stage('result-test') {
          when{
            changeset "**/result/**"
          }
        agent{
          docker {
            image 'node:8.16.0-alpine'
          }
        }

            steps {
                echo 'Running Unit Tests on result app...'
                dir('result'){
                  sh 'npm test'
                }
            }
        }
        stage('result-docker-package'){
          agent any
          when{
            changeset "**/result/**"
          }
          steps{
            echo 'Packaging result app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/','dockerlogin') 
              {
                def resultImage = docker.build("rotimi98/result:v${env.BUILD_ID}", "./result")
                resultImage.push()
                resultImage.push("${env.BRANCH_NAME}")
                resultImage.push("latest")
              }
            }
          }
        }
        stage('vote-build') {
          when{
            changeset "**/vote/**"
          }

        agent{
          docker {
            image 'python:3.11.0-bullseye'
          }
        }
            steps {
                echo 'Compiling vote app...'
                dir('vote') {
                  sh 'pip install -r requirements.txt || true'
                }
            }
        }
        stage('vote-test') {
          when{
            changeset "**/vote/**"
          }
        agent{
          docker {
            image 'python:3.11.0-bullseye'
          }
        }
            steps {
                echo 'Running Unit Tests on vote app...'
                dir('vote'){
                  sh 'nosetests -v || true'
                }
            }
        }
        stage('vote-docker-package'){
          agent any

          when{
            changeset "**/vote/**"
          }
          steps{
            echo 'Packaging vote app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/','dockerlogin') 
              {
                def resultImage = docker.build("rotimi98/vote:v${env.BUILD_ID}", "./vote")
                resultImage.push()
                resultImage.push("${env.BRANCH_NAME}")
                resultImage.push("latest")
              }
            }
          }
        }

    }
        post{
          always{
            echo 'Building multibranch pipeline for worker, result and vote is completed..'
          }

          failure{
            slackSend (channel: "training", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open)")
          }
          success{
            slackSend (channel: "training", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open)")
          }


        }
}
