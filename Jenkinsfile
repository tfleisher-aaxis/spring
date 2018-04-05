pipeline {
  agent any
  stages {
    stage('Validate') {
      parallel {
        stage('Validate') {
          steps {
            fileExists 'README.md'
            validateDeclarativePipeline 'Jenkinsfile'
            sh 'cd spring && ./mvnw validate'
          }
        }
      }
    }
    stage('Build spring') {
      agent {
        docker {
          image 'maven:3.5.3-jdk-8-alpine'
          args '-u 0:0 -v $HOME/.m2:/root/.m2'
        }
        
      }
      steps {
        sh '''cd spring
mvn --batch-mode -V -U -e clean compile test-compile -Dsurefire.useFile=false'''
        stash name: 'spring-target-build', includes: 'spring/target/**'
      }
    }
    stage('Test spring') {
      agent {
        docker {
          image 'maven:3.5.3-jdk-8-alpine'
          args '-u 0:0 -v $HOME/.m2:/root/.m2'
        }
        
      }
      steps {
        echo 'Test Stage'
        unstash name:'spring-target-build'
        //sh '''cd spring
//mvn --batch-mode -V -U -e test -Dsurefire.useFile=false -Dmaven.test.failure.ignore=true'''
        //stash name: 'testResults', includes: '**/target/surefire-reports/**/*.xml'
        echo 'No tests to run'
        stash name: 'spring-target-test', includes: 'spring/target/**'
      }
    }
    stage('Record Spring Tests') {
      agent any
      steps {
        //unstash name: 'testResults'
        //sh 'touch */target/surefire-reports/*.xml'
        //junit '**/target/surefire-reports/*.xml'
        echo 'No tests at this time'
      }
    }
    stage('Package spring') {
      agent {
        docker {
          image 'maven:3.5.3-jdk-8-alpine'
          args '-u 0:0 -v $HOME/.m2:/root/.m2'
        }
        
      }
      steps {
        echo 'Building Package'
        unstash name:'spring-target-test'
        sh '''cd spring
mvn --batch-mode -V -U -e package -DskipTests=true -Ddockerfile.skip'''
        stash(name: 'jarfiles', includes: '**/target/**/*.jar')
      }
    }
    stage('Deploy spring') {
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