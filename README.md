# Simple CI/CD Architecture for PHP developers
Simple DevOps Architecture for PHP developers using Centos, git, gitlab, Jenkins, SonarQube  
[![GitHub license](https://img.shields.io/github/license/Naereen/StrapDown.js.svg)](https://github.com/Naereen/StrapDown.js/blob/master/LICENSE)
[![Open Source Love svg1](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://github.com/ellerbrock/open-source-badges/)
[![saythanks](https://img.shields.io/badge/say-thanks-ff69b4.svg)](https://saythanks.io/to/gouzal)  

# prior
In this tutorial, we will install the latest version of SonarQube on CentOS 7.
## Install SonarQube on CentOS 7
SonarQube is an open source tool for quality system development. It is written in Java and supports multiple databases. It provides capabilities to continuously inspect code, show the health of an application, and highlight newly introduced issues. It contains code analyzers which are equipped to detect tricky issues. It also integrates easily with DevOps.

### Prerequisites
- CentOS 7 64-bit server instance with at least 2 GB RAM.
- A sudo user.

### Step 1: Perform a system update
Before installing any packages on the CentOS server instance, it is recommended to update the system.   
Log in using the sudo user and run the following commands to update the system.

```bash
sudo yum -y install epel-release
sudo yum -y update
sudo shutdown -r now
Once the system has finished rebooting, log in again as the sudo user and proceed to the next step.
```
### Step 2: Install Java
Download the Oracle SE JDK RPM package by typing:
```bash
wget --no-cookies --no-check-certificate --header "Cookie:oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm"
```
Install the downloaded package by typing:
```
sudo yum -y localinstall jdk-8u131-linux-x64.rpm
```
You can now check the version of Java by typing:
```
java -version
```
### Step 3: Install and configure PostgreSQL
Install PostgreSQL repository by typing:
```
sudo rpm -Uvh https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
```
Install PostgreSQL database server by running:
```
sudo yum -y install postgresql96-server postgresql96-contrib
```
Initialize the database:
```
sudo /usr/pgsql-9.6/bin/postgresql96-setup initdb
```
Edit the `/var/lib/pgsql/9.6/data/pg_hba.conf` to enable MD5-based authentication.
```
sudo nano /var/lib/pgsql/9.6/data/pg_hba.conf
```
Find the following lines and change `peer` to `trust` and `idnet` to `md5`.
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident  
```

Once updated, the configuration should look like the one shown below.
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5  
```

Start PostgreSQL server and enable it to start automatically at boot time by running:
```
sudo systemctl start postgresql-9.6
sudo systemctl enable postgresql-9.6
```
Change the password for the default PostgreSQL user.
```
sudo passwd postgres
```
Switch to the `postgres` user.
```
su - postgres
```
Create a new user by typing:
```
createuser sonar
```

Switch to the PostgreSQL shell.
```
psql
```

Set a password for the newly created user for SonarQube database.
```
ALTER USER sonar WITH ENCRYPTED password 'StrongPassword';
```

Create a new database for PostgreSQL database by running:
```
CREATE DATABASE sonar OWNER sonar;
```

Exit from the psql shell:
```
\q
```
Switch back to the sudo user by running the exit command.

### Step 4: Download and configure SonarQube
Download the SonarQube installer files archive.
```
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.7.zip
```
You can always look for the link to the latest version of the application on the SonarQube download page.

Install unzip by running:
```
sudo yum -y install unzip
```
Unzip the archive using the following command.
```
sudo unzip sonarqube-6.4.zip -d /opt
```
Rename the directory:
```
sudo mv /opt/sonarqube-6.4 /opt/sonarqube
```
Open the SonarQube configuration file using your favorite text editor.
```
sudo nano /opt/sonarqube/conf/sonar.properties
```
Find the following lines.
```
#sonar.jdbc.username=
#sonar.jdbc.password=
```
Uncomment and provide the PostgreSQL username and password of the database that we have created earlier. It should look like:
```
sonar.jdbc.username=sonar
sonar.jdbc.password=StrongPassword
```
Next, find:
```
#sonar.jdbc.url=jdbc:postgresql://localhost/sonar
```
Uncomment the line, save the file and exit from the editor.

### Step 5: Configure Systemd service
SonarQube can be started directly using the startup script provided in the installer package. As a matter of convenience, you should setup a Systemd unit file for SonarQube.
```
sudo nano /etc/systemd/system/sonar.
```
Populate the file with:
```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=root
Group=root
Restart=always

[Install]
WantedBy=multi-user.target
```
Start the application by running:
```
sudo systemctl start sonar
```
Enable the SonarQube service to automatically start at boot time.
```
sudo systemctl enable sonar
```
To check if the service is running, run:
```
sudo systemctl status sonar
```
### Step 5: Configure firewall
Allow the required HTTP port through the system firewall.
```
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
```

Start the SonarQube service:
```
sudo systemctl start sonar
```

You will also need to disable SELinux:
```
sudo setenforce 0
```

SonarQube is installed on your server, access the dashboard at the following address.
```
http://[IP]:9000 or http://domain.tld:9000
```

Log in using the initial administrator account, admin and admin. You can now use SonarQube to continuously analyze the code you have written.
## Install GitLab
GitLab is a web-based DevOps lifecycle tool that provides a Git-repository manager providing wiki, issue-tracking and CI/CD pipeline features, using an open-source license, developed by GitLab Inc.
### Step 1: Install and configure the necessary dependencies
On CentOS 7 (and RedHat/Oracle/Scientific Linux 7), the commands below will also open HTTP and SSH access in the system firewall.
```
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld
```
Next, install Postfix to send notification emails. If you want to use another solution to send emails please skip this step and configure an external SMTP server after GitLab has been installed.
```
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```
During Postfix installation a configuration screen may appear. Select 'Internet Site' and press enter. Use your server's external DNS for 'mail name' and press enter. If additional screens appear, continue to press enter to accept the defaults.
### Step 2: Add the GitLab package repository and install the package
Add the GitLab package repository.
```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```

Next, install the GitLab package. Change https://gitlab.example.com to the URL at which you want to access your GitLab instance. Installation will automatically configure and start GitLab at that URL.

For https:// URLs GitLab will automatically request a certificate with Let's Encrypt, which requires inbound HTTP access and a valid hostname. You can also use your own certificate or just use http://.
```
sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ee
```
Or you can edit directly the GitLab config file:
```
sudo nano /etc/gitlab/gitlab.rb
```
change the URL to your `domain` or `IP` and add the `Port` you want to use to access to the git lab.  
Note : change `https` to `http` if you don't want to use `HTTPS`
```
external_url 'http://207.180.197.78:8081'
```
and add :
```
nginx['redirect_http_to_https'] = false
```
and uncomment this line to enable which is a Rack HTTP server to serve Ruby web applications on UNIX environment. It is optimised to be used with nginx:
```
unicorn['port'] = 8083
```


### Step 3: Browse to the hostname and login
GitLab is installed on your server, access at the following address.
```
http://[IP]:8081 or http://domain.tld:8081
```
On your first visit, you'll be redirected to a password reset screen. Provide the password for the initial administrator account and you will be redirected back to the login screen. Use the default account's username root to login.

## Install Jenkins
Jenkins is an open source, Java-based automation server that offers an easy way to set up a continuous integration and continuous delivery (CI/CD) pipeline.

Continuous integration (CI) is a DevOps practice in which team members regularly commit their code changes to the version control repository, after which automated builds and tests are run. Continuous delivery (CD) is a series of practices where code changes are automatically built, tested and deployed to production.

This tutorial, will walk you through the steps of installing Jenkins on a CentOS 7 system using the official Jenkins repository
### Prerequisites
Before continuing with this tutorial, make sure you are logged in as a user with sudo privileges.

### Step 1: Install Java
Jenkins is a Java application, so the first step is to install Java.
Note: we have already installed it in previous steps.

### Step 2: enable Jenkins repository
The next step is to enable the Jenkins repository. To do that, import the GPG key using the following curl command:
```
curl --silent --location http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo | sudo tee /etc/yum.repos.d/jenkins.repo
```
And add the repository to your system with:
```
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
```
### Step 3: Installation

Once the repository is enabled, install the latest stable version of Jenkins by typing:
```
sudo yum install jenkins
```
After the installation process is completed, start the Jenkins service with:
```
sudo systemctl start jenkins
```
To check whether it started successfully run:
```
systemctl status jenkins
```
You should see something similar to this:
```
● jenkins.service - LSB: Jenkins Automation Server
Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
Active: active (running) since Thu 2018-09-20 1421 UTC; 15s ago
    Docs: man:systemd-sysv-generator(8)
Process: 2367 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)
CGroup: /system.slice/jenkins.service
```
Finally enable the Jenkins service to start on system boot.
```
sudo systemctl enable jenkins
```
output
```
jenkins.service is not a native service, redirecting to /sbin/chkconfig.
Executing /sbin/chkconfig jenkins on
```
### Step 4: Adjust the Firewall
If you are installing Jenkins on a remote CentOS server that is protected by a firewall you need to port 8080.

Use the following commands to open the necessary port:

```
sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
sudo firewall-cmd --reload
```
### Step 5: now you can access 
To set up your new Jenkins installation, open your browser and type your domain or IP address followed by port 8080:
```
http://your_ip_or_domain:8080
```
A screen  will appear, so just follow the instructions to complete Jenkins setup.

### Finally
 navigate to  [Jenkins documentation page]( https://jenkins.io/doc/ "Jenkins documentation page") and start exploring Jenkins’s workflow and plug-in model.


----

[![GitHub license](https://img.shields.io/github/license/Naereen/StrapDown.js.svg)](https://github.com/Naereen/StrapDown.js/blob/master/LICENSE)
[![Open Source Love svg1](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://github.com/ellerbrock/open-source-badges/)
[![saythanks](https://img.shields.io/badge/say-thanks-ff69b4.svg)](https://saythanks.io/to/gouzal)


**Free Software, Hell Yeah!**


[![ForTheBadge built-with-love](http://ForTheBadge.com/images/badges/built-with-love.svg)](https://GitHub.com/Naereen/)  

