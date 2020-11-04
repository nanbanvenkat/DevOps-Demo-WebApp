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
                  deploy adapters: [tomcat8(url: 'http://52.252.0.150:8080/', credentialsId: 'tomcat', path: '' )], contextPath: '/QAWebapp', war: '**/*.war'
	            }
                 }
		 stage('Deploy Artifactory') {
                 steps {
			    	id: 'Artifactory-1',
   				 url: 'https://venkatdevops.jfrog.io/artifactory',
    
    				credentialsId: 'artifactory',
    				// If Jenkins is configured to use an http proxy, you can bypass the proxy when using this Artifactory server:
    				bypassProxy: true,
    				// Configure the connection timeout (in seconds).
    				// The default value (if not configured) is 300 seconds:
    				timeout: 300             	
            	}
               }
              	               
        }
}
