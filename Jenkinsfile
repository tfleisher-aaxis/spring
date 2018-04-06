pipeline {
  agent any
  stages {
    stage('Validate') {
      parallel {
        stage('Validate sample_for_k8s') {
          agent {
            docker {
              image 'openjdk:8-jdk-alpine'
              args '-u 0:0 -v $HOME/.m2:/root/.m2'
            }
          }
          steps {
            fileExists 'README.md'
            sh 'cd sample_for_k8s && ./mvnw validate'
          }
        }
        stage('Validate pipeline') {
          steps {
            validateDeclarativePipeline 'Jenkinsfile'
          }
        }
      }
    }
    stage('Build sample_for_k8s') {
      agent {
        docker {
          image 'openjdk:8-jdk-alpine'
          args '-u 0:0 -v $HOME/.m2:/root/.m2'
        }
        
      }
      steps {
        sh '''cd sample_for_k8s
./mvnw --batch-mode -V -U -e clean compile test-compile -Dsurefire.useFile=false'''
        stash name: 'sample_for_k8s-target-build', includes: 'sample_for_k8s/target/**'
      }
    }
    stage('Test sample_for_k8s') {
      agent {
        docker {
          image 'openjdk:8-jdk-alpine'
          args '-u 0:0 -v $HOME/.m2:/root/.m2'
        }
        
      }
      steps {
        echo 'Test Stage'
        unstash name:'sample_for_k8s-target-build'
        //sh '''cd sample_for_k8s
//./mvnw --batch-mode -V -U -e test -Dsurefire.useFile=false -Dmaven.test.failure.ignore=true'''
        //stash name: 'testResults', includes: '**/target/surefire-reports/**/*.xml'
        echo 'No tests to run'
        stash name: 'sample_for_k8s-target-test', includes: 'sample_for_k8s/target/**'
      }
    }
    stage('Record sample_for_k8s Tests') {
      agent any
      steps {
        //unstash name: 'testResults'
        //sh 'touch */target/surefire-reports/*.xml'
        //junit '**/target/surefire-reports/*.xml'
        echo 'No tests at this time'
      }
    }
    stage('Package sample_for_k8s') {
      agent {
        docker {
          image 'openjdk:8-jdk-alpine'
          args '-u 0:0 -v $HOME/.m2:/root/.m2'
        }
        
      }
      steps {
        echo 'Building Package'
        unstash name:'sample_for_k8s-target-test'
        sh '''cd sample_for_k8s
./mvnw --batch-mode -V -U -e package -DskipTests=true -Ddockerfile.skip'''
        stash(name: 'sample_for_k8s-package', includes: 'sample_for_k8s/target/**')
      }
    }
    stage('Deploy sample_for_k8s') {
      /* when {
        branch 'production'
      } */
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          input(message: 'Deploy to repository?', ok: 'Approve')
        }
        
        echo 'Deploying..'
      }
    }
    stage('Build docker image') {
      agent any
      steps {
        unstash name: 'sample_for_k8s-package'
        sh """cd sample_for_k8s 
        ./mvnw --batch-mode -V -U -e dockerfile:build -DskipTests=true -Ddocker.image.repository=tfleisher/k8s-repo -Ddocker.image.tag='spring-sample_for_k8s-b${env.BUILD_NUMBER}'
        """
      }

    }
    stage('Push docker image') {
      agent any
      steps {
        script {
          // Empty url uses public repository
          withDockerRegistry([credentialsId: 'docker-hub-creds', url: '']) {
            def myImage = docker.image("tfleisher/k8s-repo:spring-sample_for_k8s-b${env.BUILD_NUMBER}")
            myImage.push()
          }
        }
      }
    }
  }
}