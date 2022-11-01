pipeline {
  agent none
  stages {
    stage('worker-build') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Building worker app'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('worker-test') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Running Unit Tests on worker app...'
        dir(path: 'worker') {
          sh 'mvn clean test'
        }

      }
    }

    stage('worker-package') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'Packaging worker app into a jarfile'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('worker-docker-package') {
      agent any
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'Packaging worker app with docker'
        script {
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
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Compiling result app...'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('result-test') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Running Unit Tests on result app...'
        dir(path: 'result') {
          sh 'npm test'
        }

      }
    }

    stage('result-docker-package') {
      agent any
      when {
        changeset '**/result/**'
        branch 'master'
      }
      steps {
        echo 'Packaging result app with docker'
        script {
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
      agent {
        docker {
          image 'python:3.11.0-bullseye'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Compiling vote app...'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt || true'
        }

      }
    }

    stage('vote-test') {
      agent {
        docker {
          image 'python:3.11.0-bullseye'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Running Unit Tests on vote app...'
        dir(path: 'vote') {
          sh 'nosetests -v || true'
        }

      }
    }

    stage('vote-docker-package') {
      agent any
      when {
        changeset '**/vote/**'
        branch 'master'
      }
      steps {
        echo 'Packaging vote app with docker'
        script {
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

    stage('deploy to dev') {
      agent any
      when {
        branch 'master'
      steps {
        sh 'docker-compose up -d'
      }
    }

  }
  post {
    always {
      echo 'Building multibranch pipeline for worker, result and vote is completed..'
    }

    failure {
      slackSend(channel: 'training', message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open)")
    }

    success {
      slackSend(channel: 'training', message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open)")
    }

  }
}
