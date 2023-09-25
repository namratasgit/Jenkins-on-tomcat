1.	Install java- sudo apt-get install openjdk-11-jdk-headless
2.	find / -name  java  -type  f  2>/dev/null
3.	nano ~/.bashrc
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
4.	source ~/.bashrc
5.	echo $java_home  to check java home variable
6.	javac –version   OR java -version
7.	Install maven to build java applications   sudo apt install maven –y  
8.	namrataubuntu@namrataubuntu-vm:~$ mvn -version
Apache Maven 3.6.3
Maven home: /usr/share/maven
Java version: 11.0.20.1, vendor: Ubuntu, runtime: /usr/lib/jvm/java-11-openjdk-amd64
Default locale: en_IN, platform encoding: UTF-8
OS name: "linux", version: "6.2.0-32-generic", arch: "amd64", family: "unix"
OR
9.	echo $PATH  find / -name mvn -type f 2>/dev/null  ]  to check maven directory
namrataubuntu-vm@namrataubuntu:~$ /usr/share/maven/bin/mvn  -version or /usr/bin/mvn -version
a.	
	Apache Maven 3.6.3
Maven home: /usr/share/maven
Java version: 11.0.20.1, vendor: Ubuntu, runtime: /usr/lib/jvm/java-11-openjdk-amd64
Default locale: en, platform encoding: UTF-8
OS name: "linux", version: "6.2.0-1010-aws", arch: "amd64", family: "unix"

10.	Install git    sudo apt install wget git –y ,….. git version
11.	Download and install Jenkins-
	wget https://updates.jenkins-ci.org/latest/jenkins.war
	java -jar jenkins.war
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

ef720c0d555f4a4bacb2aa14e6eaf1b2

This may also be found at: /home/namrataubuntu/.jenkins/secrets/initialAdminPassword
	
NamratasJenkins
namratasjenkins123

12.	Tomcat server setup
a.	sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.80/bin/apache-tomcat-9.0.80.tar.gz
b.	groupadd tomcat
c.	sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
d.	sudo tar -xzvf  apache-tomcat-9.0.80.tar.gz -C /opt/tomcat --strip-components=1
e.	cd /opt/tomcat
f.	Changing permission for the opt/tomcat folder
	sudo chgrp -R tomcat /opt/tomcat
	sudo chmod -R g+r conf
	sudo chmod g+x conf
	chown -R tomcat webapps/ work/ temp/ logs/
g.	To check the kind of java installed and whether accessible from /opt/tomcat sudo apt install galternatives , galternatives java
h.	sudo nano server.xml - change server port to 8081 and in Jenkins ec2, add inbound rule for 8081
i.	root@Jenkins:/opt/tomcat/bin# ./startup.sh  Tomcat started
j.	find / -name context.xml  to find the context file for webapps and edit it so that users can login to our manager app from outside the localhost
k.	sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml and change as given below-
“127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1"  this address is loopback address which allows login only from local system. But we want to access from outside as well. So we do the following
<!-- <Valve classname= …… /> -->  comment the statement consisting the loopback address
Same to be repeated for sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml.
j.	./shutdown.sh and ./startup.sh and check the page. Now if we click on manage app, we will be asked for username and password. Currently we do not have any user. So we create one.
We go to conf and edit tomcat-users.xml by adding users as follows-
 <role rolename="manager-gui"/>
 <role rolename="manager-script"/>
 <role rolename="manager-jmx"/>
 <role rolename="manager-status"/>
 <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
 <user username="deployer" password="deployer" roles="manager-script"/>
 <user username="tomcat" password="s3cret" roles="manager-script"/>
	Move back to bin and shutdown restart again.
k.	To set JAVA_HOME in tomcat 
root@Jenkins:/opt/tomcat/bin# sudo nano catalina.sh
  export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
To check if set correctly 
root@Jenkins:/opt/tomcat/webapps/ROOT# sudo nano java_home.jsp 
			<%@ page language="java" %>
<html>
<head>
    				<title>JAVA_HOME</title>
</head>
<body>
    				<h1>JAVA_HOME: <%= System.getenv("JAVA_HOME") %></h1>
</body>
</html>
l.	sudo nano /etc/systemd/system/tomcat.service [from root]

[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

Environment="JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target



13.	Git

A.	Manage jenkins global tool configuration jDK add name=jdk, java_home=/usr/lib/jvm/java-11-openjdk-amd64
B.	Maven installation  Name=mvn3.6.3, version-3.6.3
C.	Git --. Name = git, path =/bin/git 
D.	Create a freestyle project(first)
E.	sCM Git[add git credentials]
F.	branches to build  */master
G.	Build steps invoke top-level maven targets maven version(3.6.3), goals – clean install
H.	Post build actions deploy-to-container  WAR/EAR files = **/*.war, Context path = /, add container – tomcat 9.*, path – http://localhost:8081/manage/test.
OR
Build steps execute shell 

#!/bin/bash

# Variables
TOMCAT_DIR="/opt/tomcat"  # Tomcat installation directory
WEBAPPS_DIR="$TOMCAT_DIR/webapps"  # Tomcat webapps directory
WAR_FILE="/home/namrataubuntu/jenkins.war"  # Path to your jenkins.war file

# Stop Tomcat (if it's running)
$TOMCAT_DIR/bin/catalina.sh stop

# Remove the existing application (if any)
rm -rf "$WEBAPPS_DIR/jenkins"

# Deploy the new WAR file
cp "$WAR_FILE" "$WEBAPPS_DIR/"

# Start Tomcat
$TOMCAT_DIR/bin/catalina.sh start
14.	Save and build
