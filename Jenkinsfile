node ('master') {
	stage('Poll') {
		checkout scm
	}
	stage('Build and Unit test'){
		sh 'mvn clean verify -DskipITs=true';
		junit '**/target/surefire-reports/TEST-*.xml'
		archive 'target/*.jar'
	}
}
