#! /usr/bin/env groovy

pipeline {

  agent {
    label 'maven'
  }
  stages {
    stage("Clone Git Repository") {
            steps {
                git(
                    url: "https://github.com/malam84/springboot-hello-world2.git",
                    branch: "master",
                    changelog: true,
                    poll: true
                )
            }
        }
    stage('Build') {
      steps {
        echo 'Building..'
        sh 'mvn clean package'
        sh "mvn install -DskipTests=true"
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {

            openshift.withCluster() { 
                openshift.withProject("helloworld-jenkins-demo") {
  
                    def buildConfigExists = openshift.selector("bc", "microservice-app").exists() 
    
                    if(!buildConfigExists){ 
                       openshift.newBuild("--name=helloworld-jenkins-demo", "--docker-image=registry.redhat.io/ubi9/openjdk-17@sha256:ef190f66e41f278e7f84644c714d3cb557d34596b1821bade9fa33e0d3400d6b", "--binary") 
                    } 
    
                    openshift.selector("bc", "microservice-app").startBuild("--from-file=target/springboot-hello-world2-0.0.1-SNAPSHOT.jar", "--follow") 
                }
            }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
            openshift.withProject("helloworld-jenkins-demo") { 
                 def deployment = openshift.selector("dc", "microservice-app") 
    
                if(!deployment.exists()){ 
                    openshift.newApp('microservice-app', "--as-deployment-config").narrow('svc').expose() 
                } 
    
                timeout(5) { 
                    openshift.selector("dc", "microservice-app").related('pods').untilEach(1) { 
                        return (it.object().status.phase == "Running") 
                    } 
                } 
            } 
          }

        }
      }
    }
  }
}
