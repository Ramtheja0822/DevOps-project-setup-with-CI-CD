# DevOps-project-setup-with-CI-CD

### DevOps project setup with CI/CD ### 
1. Maven
2. Git Hub
3. Tomcat - Tomcat Server -- Linux OS - apache-tomcat-9.0.65 installed -- start the tomcat server using 8080 port  (t2.micro)
4. SonarQube - SonarQube Server -- Linux OS - sonarqube-7.8 installed -- cd bin -- cb linux86-64/  --start sonarQube using 9000 (t2.medium)
5. Nexus Repo - NexuRepository -- login using 8081 (t2.medium)
6. Jenkins (t2.Micro)

############ Stages ################
State-1: Clone Repositiry
State-2: Maven Build
State-3: Code Review
State-4: Upload Artifact (Nexus /jfrag)
State-5: Deploy App
####################################

#### Jenkins Installation Steps ##############################################################################

1. Create Ubuntu VM Using AWS EC2
2. Install Java & Jenkins using below commands
3. sudo apt-get update
4. sudo apt-get install default-jdk
5. wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
6. sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
7. sudo apt-get update
8. sudo apt-get install jenkins
9. sudo systemctl status jenkins

To copy Jenkins default password you can use below command 

10. sudo cat /var/lib/jenkins/secrets/initialAdminPassword

##############################################################################################################

In the jenkins need to set the Maven as global tool 

#### pipeline Script ####

node{
    
    stage('Clone Repository'){
        git credentialsId: 'Git-Credentials', url: 'https://github.com/Ramtheja0822/maven-web-app.git'
     
    }
    
    stage('Maven Build'){
        def mavenHome = tool name: "Maven-3.8.6", type: "maven"
	    def mavenCMD = "${mavenHome}/bin/mvn"
	    sh "${mavenCMD} clean package"
    }
    
    stage('Code Review'){
        withSonarQubeEnv('SonarQube-9'){
		def mavenHome = tool name: "Maven-3.8.6", type: "maven"
		def mavenCMD  = "${mavenHome}/bin/mvn"
		sh "${mavenCMD} sonar:sonar"
		}
    }
    
    stage('Upload Build Artifact'){
        nexusArtifactUploader artifacts: [[artifactId: '01-maven-web-app', classifier: '', file: 'target/maven-web-app.war', type: 'war']], credentialsId: 'nexus-credentials', groupId: 'in.conprosol', nexusUrl: '44.211.203.55:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'conprosol-snapshot-repository', version: '1.0-SNAPSHOT'
    }
    
    stage('Deploy'){
        sshagent(['tomcat-server-agent']) {
         sh 'scp -o StrictHostKeyChecking=no target/maven-web-app.war ubuntu@34.226.148.201:/opt/tomcat/webapps'
        
        }
        
    }
}

########################### SonarQube configuration in Jenins ################################################
cd /opt/sonarqube-7.8/bin/linux-x86-64

sh sonar.sh start

-->Manage Jenins-> Plugins -> Available -> SonarQube scanner Plugins -> Install It

--> Generate sonar token from sonar server web URL and add it as secret text in Global credentials 

--> Manage Jenkins -> Configure System -> Sonar qube servers --> add sonarQube server

			-- Name :- Sonar-Server-7.8 (you can give any name)
			-- Server URL :- http://-----:9000/
			-- Add Sonar Server Token (Token we should add as scret text) and save it.
### Nope:- You can use below URL to configure Sonaqube

https://www.fosstechnix.com/how-to-install-sonarqube-on-ubuntu-22-04-lts/

##############################################################################################################
			Install ssh-agent plug in in jenkins
			
			ssh-agent is used to deploy war file from Jenkins to tomcat server
				
			Source : target/01-maven-web-app.war
			
			destination: /home/user/apache-tomcat-9/webapps
			
