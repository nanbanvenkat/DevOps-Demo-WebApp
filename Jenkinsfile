pipeline {
         agent any
	 tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven 'Maven'
   	 }
	 stages {
                 stage('Clone source') {
                 steps {
                      git url: 'https://github.com/nanbanvenkat/DevOps-Demo-WebApp.git'
                 	}
                 }
		 
		 stage('Maven Build') {
                 steps {
                      sh 'mvn -Dmaven.test.failure.ignore=true clean package' 		      
                 	}
                 }
		 
	//	stage('SonarQube') {
        //         steps {
        //            withSonarQubeEnv(credentialsId: 'sonar', installationName: 'sonarqube') { 
     	//		sh 'mvn clean package sonar:sonar -Dsonar.host.url=http://13.82.181.227:9000/ -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.exclusions=**/test/java/servlet/createpage_junit.java'
        //			}
	//		}
        //        }
		 
                 stage('Deploy to test') {
                 steps {
                  deploy adapters: [tomcat8(url: 'http://13.68.94.136:8080/', credentialsId: 'tomcat', path: '' )], contextPath: '/QAWebapp', war: '**/*.war'
	            }
                 }
		 
		 stage('Deploy Artifacts') {
                 steps{
			 
			rtMavenRun ( 
				tool: 'Maven',
				pom: 'pom.xml', 
				goals: 'clean install', 
				deployerId: "MAVEN_DEPLOYER", 
				resolverId: "MAVEN_RESOLVER" 
			)
				rtServer (
                    			id: 'ARTIFACTORY_SERVER',
                    			url: 'https://venkatdevops.jfrog.io/artifactory',
                    			credentialsId: 'deploy'
                			)
                	rtMavenDeployer (
                    			id: 'MAVEN_DEPLOYER',
                    			serverId: 'artifactory',
                    			releaseRepo: "libs-release-local",
                    			snapshotRepo: "libs-snapshot-local"
                			)
                	rtMavenResolver (
                    			id: 'MAVEN_RESOLVER',
                    			serverId: 'artifactory',
                    			releaseRepo: "libs-release",
                    			snapshotRepo: "libs-snapshot"
                		)
			 rtPublishBuildInfo (
				 serverId: 'artifactory'
			 )			 	
			}
		}
		 
		stage('UI-test') {
                 steps {
		 	script{
				def rtMaven = Artifactory.newMavenBuild()
				def buildInfo
				rtMaven.tool = "Maven"
				buildInfo = rtMaven.run pom: 'functionaltest/pom.xml', goals: 'test'
				publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
				}
			}
		}
		
	/*	stage('Performance Test') {
		steps{
			script{
			blazeMeterTest credentialsId: 'Blazemeter', testId: '8663167.taurus', workspaceId: '659085'
			}
		}
		}	*/
		
		 stage('Deploy to Prod') {
                 steps {
                  deploy adapters: [tomcat8(url: 'http://52.188.207.204:8080/', credentialsId: 'tomcat', path: '' )], contextPath: '/ProdWebapp', war: '**/*.war'
		  slackSend channel: 'alerts', message: "Deploy To Prod ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", teamDomain: 'venkattcsdevops', tokenCredentialId: 'slack'
	            }
		 }
		    
		stage('Sanity test') {
                 steps {
		 	script{
				def rtMaven = Artifactory.newMavenBuild()
				def buildInfo
				rtMaven.tool = "Maven"
				buildInfo = rtMaven.run pom: 'Acceptancetest/pom.xml', goals: 'test'
				publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
				}
			slackSend channel: 'alerts', message: "Sanity Test ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", teamDomain: 'venkattcsdevops', tokenCredentialId: 'slack'
			}
		}   
          	               
        }
}
