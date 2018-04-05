pipeline {
  agent any
  stages {
    stage('Validate') {
      parallel {
        stage('Validate') {
          steps {
            fileExists 'README.md'
            validateDeclarativePipeline 'Jenkinsfile'
            sh 'cd sample_for_k8s && ./mvnw validate'
          }
        }
      }
    }
    stage('Build sample_for_k8s') {
      agent {
        docker {
          image 'maven:3.5.3-jdk-8-alpine'
          args '-u 0:0 -v $HOME/.m2:/root/.m2'
        }
        
      }
      steps {
        sh '''cd sample_for_k8s
mvn --batch-mode -V -U -e clean compile test-compile -Dsurefire.useFile=false'''
        stash name: 'spring-target-build', includes: 'spring/target/**'
      }
    }
    stage('Test sample_for_k8s') {
      agent {
        docker {
          image 'maven:3.5.3-jdk-8-alpine'
          args '-u 0:0 -v $HOME/.m2:/root/.m2'
        }
        
      }
      steps {
        echo 'Test Stage'
        unstash name:'spring-target-build'
        //sh '''cd sample_for_k8s
//mvn --batch-mode -V -U -e test -Dsurefire.useFile=false -Dmaven.test.failure.ignore=true'''
        //stash name: 'testResults', includes: '**/target/surefire-reports/**/*.xml'
        echo 'No tests to run'
        stash name: 'spring-target-test', includes: 'spring/target/**'
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
          image 'maven:3.5.3-jdk-8-alpine'
          args '-u 0:0 -v $HOME/.m2:/root/.m2'
        }
        
      }
      steps {
        echo 'Building Package'
        unstash name:'spring-target-test'
        sh '''cd sample_for_k8s
mvn --batch-mode -V -U -e package -DskipTests=true -Ddockerfile.skip'''
        stash(name: 'jarfiles', includes: '**/target/**/*.jar')
      }
    }
    stage('Deploy sample_for_k8s') {
      /* when {
        branch 'production'
      } */
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          input(message: 'Deploy?', ok: 'Approve')
        }
        
        echo 'Deploying..'
      }
    }
  }
}