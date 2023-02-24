pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes

  tools{
    maven 'my_maven'
  }

  environment {
    gitName = 'oolr'
    gitEmail = 'jyy013@gmail.com'
    gitWebaddress = 'https://github.com/oolr/sb_code.git'
    gitSshaddress = 'git@github.com:oolr/sb_code.git'
    gitCredential = 'git_cre' //github credential 생성시의 ID
    dockerHubRegistry = 'jyy0103/sbimage'
    dockerHubRegistryCredential = 'docker_cre'
  }


  stages {
    stage('Checkout Github') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: gitCredential, url: gitWebaddress]]])
      }
      post {
        failure {
            echo 'Repository clone failure'
        }
        success {
            echo 'Repository clone success'
        }
     }
    }

    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
        // maven 플러그인이 미리 설치 되어 있어야함
      }
      post {
        failure {
            echo 'Maven build failure'
        }
        success {
            echo 'Maven build success'
        }
     }
    }

    stage('Docker image Build') {
      steps {
        sh "docker build -t ${dockerHubRegistry}:${currentBuild.number} ."
        sh "docker build -t ${dockerHubRegistry}:latest ."
        // jyy01-3/sbimage:4  이런식으로 빌드가 될 것이다.
        // currentBuild.number 젠킨스에서 제공하는 빌드넘버변수.
      }
      post {
        failure {
            echo 'Docker image build failure'
        }
        success {
            echo 'Docker image build success'
        }
     }
    }

    stage('Docker image Push') {
      steps {
        withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {
          // withDockerRegistry : docker pipeline 플러그인 설치시 사용가능.
          // dockerHubRegistryCredential : environment에서 선언한 docker_cre  
            sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry}:latest"
          }
      }
      post {
        failure {
            echo 'Docker image push failure'
        }
        success {
            echo 'Docker image push success'
        }
     }
    }

    stage('Docker container deploy') {
      steps {       
            sh "docker rm -f sb"
            sh "docker run -dp 5656:8085 --name sb ${dockerHubRegistry}:${currentBuild.number}"
          }
      }
      post {
        failure {
            echo 'Docker container deploy failure'
        }
        success {
            echo 'Docker container deploy success'
        }
     }

    }
  }
