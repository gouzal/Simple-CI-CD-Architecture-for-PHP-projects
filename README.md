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
wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-6.4.zip
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

----

[![GitHub license](https://img.shields.io/github/license/Naereen/StrapDown.js.svg)](https://github.com/Naereen/StrapDown.js/blob/master/LICENSE)
[![Open Source Love svg1](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://github.com/ellerbrock/open-source-badges/)
[![saythanks](https://img.shields.io/badge/say-thanks-ff69b4.svg)](https://saythanks.io/to/gouzal)


**Free Software, Hell Yeah!**


[![ForTheBadge built-with-love](http://ForTheBadge.com/images/badges/built-with-love.svg)](https://GitHub.com/Naereen/)  

