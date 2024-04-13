Create ec2 node for jenkins server select amazon linux
===============================Jenkins Installation==================================
https://www.jenkins.io/doc/book/installing/linux/
https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/
$ sudo su
$ sudo yum update –y
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
$ sudo yum upgrade
$ amazon-linux-extras install epel
$ sudo amazon-linux-extras install java-openjdk11 -y
$ yum install java-11-amazon-corretto -y
$ sudo yum install jenkins -y
$ sudo systemctl enable jenkins       //Enable the Jenkins service to start at boot
$ sudo systemctl start jenkins        //Start Jenkins as a service
$ java -version
$ javac -version
$ systemctl status jenkins
To change hostname
$ hostname JENKINS-SERVER
$ cd /etc
$ vi hostname
Add 8080 port in security group to run jenkins
===============================Install and Configure Maven==================================
https://maven.apache.org/install.html
Copy the download link from https://maven.apache.org/download.cgi
$ sudo su  & cd ~
$ cd /opt
$ wget https://dlcdn.apache.org/maven/maven-3/3.9.3/binaries/apache-maven-3.9.3-bin.tar.gz
$ tar -xzvf apache-maven-3.9.3-bin.tar.gz
$ ls
$ mv apache-maven-3.9.3 maven
$ ll
$ cd maven
$ cd bin/
$ ./mvn -v  
$ cd ~
$ pwd
$ ll -a      //It will show the hidden files also
// setting up env variable
$ vim .bash_profile
$ find / -name java-11*
//enter below lines below the 2nd fi
M2_HOME=/opt/maven
M2=/opt/maven/bin
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.19.0.7-1.amzn2.0.1.x86_64
PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2
$ echo $PATH
$ source .bash_profile
$ echo $PATH
$ mvn -v
// In jenkins GUI
// Install Maven integration plugin
// Go to tools 
// under JDK
// Name -java11
Java Path -- /usr/lib/jvm/java-11-openjdk-11.0.19.0.7-1.amzn2.0.1.x86_64
// under maven
MAVEN_HOME:/opt/maven     //You need to add this at Jenkins Job under Maven Installations
// Enable github plugin
// Restart jenkins
// Now go ec2 machine
$ yum install git
// Create new item - test-maven-project
// Select source code as git
// branch main
// under root POM
clean install
// Click build now

====================================Create Tomcat Server========================================
Create new ec2 , amazon linux-2 HVM
Add port 8080 for tomcat server in the security group
$ sudo su
$ cd ~
$ sudo amazon-linux-extras install java-openjdk11 -y
$ wget copy tar gz link from tomcat downloads
$ tar -xzvf file.tar.gz
$ mv source tomcat
$ cd bin
to start tomcat
$ ./startup.sh
$ cd ..
$ find / -name context.xml
vim to hostmanager file 
Comment that <value /> part
and in the manager file comment similarly
$ cd /bin
$ ./shutdown.sh
$ ./startup.sh
$ cd ..
$ cd conf/
$ vim tomcat-users.xml
Shift + G (go to end of the file)
replace this with the last 
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="admin" password="admin" roles="manager-gui,manager-script,manager-jmx,manager-status" />
<user username="deployer" password="deployer" roles="manager-script" />
<user username="tomcat" password="s3cret" roles="manager-gui" />

$ cd ..
$ cd /bin
$ ./shutdown.sh
$ ./startup.sh

In tomcat ip in browser
Go to  Manager APP

Integrate tomcat with Jenkins

Go to jenkins GUI and install plugin -  deploy to container
Add credentials of tomcat server to jenkins
Go to credentials
Add credential
Username - deployer
give pass and desc

Create a new Job build and deply to tomcat
Give git repo
under POM
clean install
Under post build actions
deploy war container
**/*.war
Under container add tomcat 8x
credentials tomcat credentials
Provide url of tomcat server

===============================Install and Configure Docker Host==================================

Create new ec2 instance ubuntu
$ sudo apt update && sudo apt-get update
$ sudo apt install docker.io
$ sudo usermod -aG docker ubuntu (Give complete access to username ubuntu)
$ sudo nano /etc/hostname
$ sudo init 6 (restart the server)
$ chown -R ubuntu:ubuntu /opt
$ chown -R ubuntu:ubuntu /opt/Docker
$ cd /opt/Docker
$ nano Dockerfile      //Below is the content of Dockerfile
FROM tomcat:latest
RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
COPY ./*.war /usr/local/tomcat/webapps
$ docker build -t webapp:v1 .
$ docker stop registerapp
$ docker rm registerapp
$ docker run -d --name registerapp -p 8086:8080 webapp:v1

