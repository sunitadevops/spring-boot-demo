pipeline {
  agent any

  tools {
    jdk 'jdk'
    maven 'M3'
  }

  stages {
    stage('Build') {
      steps {
        withMaven(maven : 'M3') {
          sh "mvn package"
        }
      }
    }

    stage ('OWASP Dependency-Check Vulnerabilities') {
      steps {
        withMaven(maven : 'M3') {
          sh 'mvn dependency-check:check'
        }

        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      }
    }

    stage('SonarQube analysis') {
      steps {
        withSonarQubeEnv(credentialsId: 'sonar', installationName: 'sonarqube') {
          withMaven(maven : 'M3') {
            sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html'
          }
        }
      }
    }

    stage('Create and push container') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
          withMaven(maven : 'M3') {
            sh "mvn jib:build"
          }
        }
      } 
    }

    stage('Anchore analyse') {
      steps {
        writeFile file: 'anchore_images', text: 'docker.io/sunita95/spring-boot-demo'
        anchore name: 'anchore_images'
      }
    }

    stage('Deploy to K8s') {
      steps {
        script {
        //kubernetesDeploy(configs: "k8s.yml", kubeconfigId: "kubernetes-config")
        //{
          sh 'kubectl --kubeconfig=$JENKINS_HOME/kubeconfig create -f $WORKSPACE/k8s.yml'
       // }
       }
      } 
    }
  }
}
