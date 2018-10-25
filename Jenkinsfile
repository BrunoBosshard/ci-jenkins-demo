environment { 
	CRED = credentials('jenkins-account') 
}
node('master') {
	stage('Poll') {
		checkout scm
		env.POMPATH = "${env.WORKSPACE}"
	}
	stage('Build and Unit test'){
		sh 'mvn clean verify -DskipITs=true';
		junit '**/target/surefire-reports/TEST-*.xml'
		archive 'target/*.jar'
	}
	stage('SonarQube Scan') {
		node {
			withSonarQubeEnv('Default SonarQube server') {
				sh 'mvn clean verify -f $POMPATH/pom.xml sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example-project -Dsonar.projectVersion=$BUILD_NUMBER';
			} // SonarQube taskId is automatically attached to the pipeline context
		}
	}
	stage('Quality Gate') {
//		steps {
//			script {
//				while(true) {
//					sh 'sleep 2'
//					def url="http://http://jenkins-pepgo.ngrok.io/job/${env.JOB_NAME.replaceAll('/','/job/')}/lastBuild/consoleText";
//					def sonarId = sh script: "wget -qO- --content-on-error --no-proxy --auth-no-challenge --http-user=${CRED_USR} --http-password=${CRED_PSW} '${url}'  | grep 'More about the report processing' | head -n1 ",returnStdout:true
//					sonarId = sonarId.substring(sonarId.indexOf("=")+1)
//					echo "sonarId ${sonarId}"
//					def sonarUrl = "http://sonarqube-pepgo.ngrok.io/api/ce/task?id=${sonarId}"
//					def sonarStatus = sh script: "wget -qO- '${sonarUrl}' --no-proxy --content-on-error | jq -r '.task' | jq -r '.status' ",returnStdout:true
//					echo "Sonar status ... ${sonarStatus}"
//					if(sonarStatus.trim() == "SUCCESS"){
//						echo "BREAK";
						break;
//					}
//					if(sonarStatus.trim() == "FAILED "){
//						echo "FAILED"
//						currentBuild.result = 'FAILED'
//						break;
//					}
//				}
//			}
//		}
//	timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
//			def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
//			if (qg.status != 'OK') {
//				error "Pipeline aborted due to quality gate failure: ${qg.status}"
//			}
//		}
	}
	stage ('Integration Test'){
		sh 'mvn clean verify -Dsurefire.skip=true';
		junit '**/target/failsafe-reports/TEST-*.xml'
		archive 'target/*.jar'
	}
	stage ('Publish'){
		def server = Artifactory.server 'Default Artifactory Server'
		def uploadSpec = """{
			"files": [
				{
					"pattern": "target/hello-0.0.1.war",
					"target": "example-project/${BUILD_NUMBER}/",
					"props": "Integration-Tested=Yes;Performance-Tested=No"
				}
			]
		}"""
		server.upload(uploadSpec)
	}
}
