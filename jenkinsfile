node {
    try{
    // Get Artifactory server instance, defined in the Artifactory Plugin administration page
    
    	def server = Artifactory.server "artifactory"
    
    // Create an Artifactory Maven instance.
    
    	def rtMaven = Artifactory.newMavenBuild()
    	def buildInfo 
	    
      
	stage('Clone source') {
        git url: 'https://github.com/Umamahesh-DevOpsPjt/DevOps-Demo-WebApp.git'
    }
	    
	stage('Artifactory Configuration') {  
        rtMaven.tool = "maven"   
        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
	    }
	//slackSend channel: 'tcsdevopstalk', message: "started ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slack'
	    
   
    
  stage('SonarQube') {
       withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'sonarqube') { 
     	sh 'mvn clean package sonar:sonar -Dsonar.host.url=http://34.72.139.131:9000/ -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.exclusions=**/test/java/servlet/createpage_junit.java'
        }
  }
	  
  stage('Quality Gate'){	  	  
	timeout(time: 1, unit: 'HOURS') { // If something goes wrong, pipeline will be killed after a timeout
	    def qualitygate = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
	    if (qualitygate.status != 'OK') {
	      error "Pipeline aborted due to quality gate failure: ${qualitygate.status}"
	    }
	}  
  }
  
    
    stage('Maven build') {
       buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean package', buildInfo: buildInfo
    }

    stage('Deploy to Test') {
	deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://104.197.11.170:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
	//jiraSendDeploymentInfo environmentId: 'Test', environmentName: 'QA test', environmentType: 'testing', serviceIds: ['http://104.197.11.170:8080/QAWebapp/'], site: 'devopsbc.atlassian.net', state: 'successful'
    }

    stage('Publish build info') {
        server.publishBuildInfo buildInfo
    }

    stage('UI Test') {
        buildInfo = rtMaven.run pom: 'functionaltest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
   }
   
    stage('Performance Test') {
    	echo 'Running BlazeMeterTest' 
    }

    stage('Deploy to Prod') {
	      deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://35.239.93.241:8080/')], contextPath: '/ProdWebapp', war: '**/*.war'
	     //jiraSendDeploymentInfo environmentId: 'Staging', environmentName: 'Staging', environmentType: 'staging', serviceIds: ['http://35.239.93.241:8080/ProdWebapp'], site: 'devopsbc.atlassian.net', state: 'successful'
	     //jiraSendDeploymentInfo environmentId: 'Prod', environmentName: 'prod', environmentType: 'production', serviceIds: ['http://35.239.93.241:8080/ProdWebapp'], site: 'devopsbc.atlassian.net', state: 'successful'
        }

    stage('Sanity Test') {
        buildInfo = rtMaven.run pom: 'Acceptancetest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
    }
	//slackSend channel: 'tcsdevopstalk', message: "Build Completed ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slack'
 }
 catch (exc) {
 	echo 'I failed'
	//slackSend channel: 'tcsdevopstalk', message: "Build Failed ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slack'
 }
 finally {
	if (currentBuild.result == 'failure') {
            echo 'Build is unstable'
	     error " failed"
        } else {
		echo 'Build is stable, Build Successfully ${env.BUILD_NUMBER}'
        }
 }
}
