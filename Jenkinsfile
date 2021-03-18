

pipeline {
    agent {
        label "master"
    }
    tools {
        maven "Maven"
    }
	stages {
		stage("Check Out") {
			steps {
				script {
					checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'Nexus_Cred', url: 'https://github.com/sessiondevops/et2.git']]])	
				}
			}
		}
		stage("Build") {
			steps {
				script {
					sh 'mvn clean install'
				}
			}
		} 
		stage("Nexus Upload") {
			steps {
				script {
					def pom = readMavenPom file: ''
					def nexusRepoName = pom.version.endsWith("SNAPSHOT") ? "et2-snapshot" : "et2-release"
					//echo  "${projectArtifactId} ${projectVersion}"
					nexusArtifactUploader artifacts: [
						[
							artifactId: "${pom.artifactId}", 
							classifier: '', 
							file: "target/${pom.artifactId}-${pom.version}.war", 
							type: 'war'
						]
					], 
						credentialsId: 'Nexus_Cred', 
						groupId: 'com.marsh', 
						nexusUrl: 'ec2-3-140-210-82.us-east-2.compute.amazonaws.com:8081', 
						nexusVersion: 'nexus3', 
						protocol: 'http', 
						repository: nexusRepoName, 
						version: "${pom.version}"
                }				
                    
            }
		}
		stage("Pull and deploy") {
			steps {
				script {
					sh 'curl http://ec2-3-140-210-82.us-east-2.compute.amazonaws.com:8081/repository/et2-snapshot/com/marsh/et2/0.0.3-SNAPSHOT/et2-0.0.3-20210318.072233-1.war -o /usr/share/apache-tomcat-9.0.39/webapps/${pom.artifactId}.war'
					sh 'chown -R jenkins:jenkins /usr/share/apache-tomcat-9.0.39/webapps/${pom.artifactId}.war',
					sh '/usr/share/apache-tomcat-9.0.39/bin/startup.sh'
				}
			}
		} 
    }   
	/*post {
        always {
            deleteDir() /* clean up our workspace 
        }
    }*/
}
