def mvn
def server = Artifactory.server 'artifactory'
def rtMaven = Artifactory.newMavenBuild()
def buildInfo
def DockerTag() {
	def tag = sh script: 'git rev-parse HEAD', returnStdout:true
	return tag
	}
pipeline {
  agent { label 'master' }
    tools {
      maven 'Maven'
      jdk 'JAVA_HOME'
    }
  options { 
    timestamps () 
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '5')	
// numToKeepStr - Max # of builds to keep
// daysToKeepStr - Days to keep builds
// artifactDaysToKeepStr - Days to keep artifacts
// artifactNumToKeepStr - Max # of builds to keep with artifacts	  
}	
  environment {
    SONAR_HOME = "${tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'}"
    DOCKER_TAG = DockerTag()	  
  }  
  stages {
    stage('Artifactory_Configuration') {
      steps {
// 	      sh 'mvn dependency:purge-local-repository'
        script {
		  rtMaven.tool = 'Maven'
		  rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
		  buildInfo = Artifactory.newBuildInfo()
		  rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot', server: server
          buildInfo.env.capture = true
        }			                      
      }
    }
    stage('Execute_Maven') {
	  steps {
		  sh 'mvn clean install'
// 	    script {
// 		  rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
//         }			                      
      }
    }	
    stage('War rename') {
	  steps {
		  	sh 'mv target/*.war target/helloworld.war'
        }			                      
      }
    stage('SonarQube_Analysis') {
      steps {
	    script {
          scannerHome = tool 'sonar-scanner'
        }
        withSonarQubeEnv('sonar') {
      	  sh """${scannerHome}/bin/sonar-scanner"""
        }
      }	
    }	
	
  stage('Build Docker Image'){
    steps{
      sh 'sudo docker build -t dileep95/ansibledeploy:${DOCKER_TAG} .'
    }
  }
stage('Build Docker Image'){
    steps{
      sh 'sudo docker run -dit dileep95/ansibledeploy:${DOCKER_TAG} '
    }
  }
     	  
  stage('test'){
      steps {
       //ansiblePlaybook credentialsId: 'ans-server', inventory: 'inventory', playbook: 'ansibleplay.yml', tags: 'stop_container,delete_container'
       // Above command used to run the playbook with specified tags mentioned in the tags section.	
	 sh 'sudo docker ps '
        }
      }	  	  
  }
}   
