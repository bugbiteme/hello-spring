pipeline {
  agent {
      label 'maven'
  }
  stages {
    stage('Build JAR') {
      steps {
        sh "mvn -B -DskipTests clean package"
        stash name:"jar", includes:"target/hello-0.0.1-SNAPSHOT.jar"
      }
    }
    stage('Test') {
      steps {
        sh 'mvn test'
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
        }
      }
    }
    stage('Build Image') {
      steps {
        unstash name:"jar"
        script {
          openshift.withCluster() {
            openshift.startBuild("hello", "--from-file=target/hello-0.0.1-SNAPSHOT.jar", "--wait")
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
          openshift.withCluster() {
			def dc = openshift.selector("dc", "hello")
			dc.rollout().latest()
            dc.rollout().status()
          }
        }
      }
    }
  }
}
