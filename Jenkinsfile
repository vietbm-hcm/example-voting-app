pipeline {
  agent none
  stages{
    stage('worker build'){
      agent { 
        docker{
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      when{
        changeset "**/worker/**"
      }
      steps{
        echo 'Compiling worker app'
        dir('worker'){
          sh 'mvn compile'
        }
      }
    }
    stage('worker test'){
      agent { 
        docker{
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      when{
        changeset "**/worker/**"
      }
      steps{
        echo 'Running Unit tests on worker app'
        dir('worker'){
          sh 'mvn clean test'
        }
      }
    }
    stage('worker docker-package'){
      agent any
      when{
        branch 'master'
        changeset "**/worker/**"
      }
      steps{
        echo 'Packing worker app with docker'
        script{
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
            def workerImage = docker.build("minhviet/worker:v${env.BUILD_ID}", "./worker/")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }
      }
    }
    stage('Vote install'){
      agent{
        docker{
          image 'python:3.7.7-alpine3.11'
          args '--user root'
        }
      }
      when{
        changeset '**/vote/**'
      }
      steps{
        echo 'Intall package'
        dir('vote'){
          sh 'pip install -r requirements.txt'
        }
      }
    }
    stage('Vote Unit test'){
      agent{
        docker{
          image 'python:3.7.7-alpine3.11'
          args '--user root'
        }
      }
      when{
        changeset '**/vote/**'
      }
      steps{
        dir('vote'){
          echo 'Unit test'
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }
      }
    }
    stage('Vote push Image docker'){
      agent any
      when{
        branch 'master'
        changeset '**/vote/**'
      }
      steps{
        echo "Push image vote to registry docker"
        script{
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
            def resultImage = docker.build("minhviet/vote:v${env.BUILD_ID}", "./vote/")
            resultImage.push()
            resultImage.push("${env.BRANCH_NAME}")
          }
        }
      }
    }
    stage('Result build'){
      agent{
        docker{
          image 'node:12.18.1-alpine3.9'
        }
      }
      when{
        changeset "**/result/**"
      }
      steps{
        echo "Compile result app"
        dir('result'){
          sh 'npm install'
        }   
      }
    }
    stage('Result test'){
      agent{
        docker{
          image 'node:12.18.1-alpine3.9'
        }
      }
      when{
        changeset "**/result/**"
      }
      steps{
        echo "Running unit test on result app"
        dir('result'){
          sh 'npm install'
          sh 'npm test'
        }     
      }
    }
    stage('Result push Image docker'){
      agent any
      when{
        branch 'master'
        changeset '**/result/**'
      }
      steps{
        echo "Push image result to registry docker"
        script{
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
            def resultImage = docker.build("minhviet/result:v${env.BUILD_ID}", "./result/")
            resultImage.push()
            resultImage.push("${env.BRANCH_NAME}")
          }
        }
      }
    }
    stage('Sonarqube') {
        agent any
        environment{
          sonarpath = tool 'SonarScanner'
        }
        steps {
            echo 'Running Sonarqube Analysis..'
              withSonarQubeEnv('sonar') {
                sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
              }
        }
    }
    stage('deploy to dev'){
      agent any
      when{
        branch 'master'
      }
      steps{
        echo 'Deploy instavote app with docker compose'
        sh '/usr/local/bin/docker-compose down -v --rmi all'
        sh '/usr/local/bin/docker-compose up -d'
      }
    }
  }
  post {
    always{
      echo 'Pipeline for instavote app successfully'
    }
    failure{
      slackSend (channel: "general", message: "Build failed- ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }
    success{
      slackSend (channel: "general", message: "Build successed- ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }
  }
}