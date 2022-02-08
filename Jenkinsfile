def mvn
def buildInfo
pipeline {
  agent { label 'master' }
    tools {
      maven 'Maven'
      jdk 'JAVA_HOME'
    }

  environment {
    SONAR_HOME = "${tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'}"	  
  }  
  stages {
    stage('Execute_Maven') {
	  steps {
		  sh 'mvn clean install'		                      
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
          scannerHome = tool 'sonar'
        }
        withSonarQubeEnv('sonar') {
      	  sh """${scannerHome}/bin/sonar-scanner"""
        }
      } 
    }
    stage('Artifacts') {
      steps {     
		rtServer (
		    id: 'jfrog',
		    timeout: 300
		)
            }
        }
    stage('Artifacts upload') {
      steps {     
		rtUpload (
    serverId: 'jfrog',
    spec: '''{
          "files": [
            {
              "pattern": "target/*.war",
              "target": "example-repo-local/"
            }
         ]
    }'''
)
            }
        }
     stage('ecr push') {
      steps {     
		sh 'aws ecr get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin 754733740943.dkr.ecr.us-east-1.amazonaws.com'
	        sh 'sudo docker build -t myrepo:v${BUILD_NUMBER} .'
	        sh 'sudo docker tag myrepo:v${BUILD_NUMBER} 754733740943.dkr.ecr.us-east-1.amazonaws.com/myrepo:v${BUILD_NUMBER}'
	        sh 'sudo docker push 754733740943.dkr.ecr.us-east-1.amazonaws.com/myrepo:latest'
            }
        }
    stage('Deploying container in kubernet') {
      steps {     
		sh 'sudo kubectl create deploy myweb-v${BUILD_NUMBER} --image 754733740943.dkr.ecr.us-east-1.amazonaws.com/myrepo:v${BUILD_NUMBER}'

            }
        }
   
  }
}   
